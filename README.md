# AR-VR: CNN Visualizer

An interactive **VR/AR visualization of a Convolutional Neural Network (CNN)**. A Python backend runs a PyTorch CNN trained on MNIST digits, captures live layer activations, and streams them over a WebSocket to a Unity XR project, which renders the feature maps, convolution filters, and fully-connected layer graph as 3D objects a user can inspect in headset.

## How it works

```
Unity (draw a digit on an in-VR whiteboard)
        │  exports PNG to Assets/Pics/
        ▼
Python server (server.py)
        │  loads latest PNG → preprocesses to 28x28 MNIST-style image
        │  runs it through the CNN (model.py) with forward hooks on conv/pool layers
        │  encodes activations + FC layer weights as base64 PNG/float arrays
        ▼
WebSocket (ws://localhost:8765)
        ▼
Unity client (CNNWebSocketClient.cs / WSManager.cs)
        │  deserializes into DataModels.cs (FeatureMapData, WebSocketData)
        ▼
Visualization (CNNBlockAnimator.cs, FCNetworkVisualizer.cs, ImageCubeSpawner.cs)
   renders conv feature maps as image cubes and the FC layers as a connected node graph
```

## Repository structure

```
AR-VR/
├── server/                  # Python backend
│   ├── model.py              # CNN architecture, activation hooks, helper utilities
│   ├── server.py             # WebSocket server: preprocesses input, runs inference, streams results
│   ├── server_realtime.py    # Real-time variant of the server
│   ├── receiver.py           # Companion receiver script
│   ├── requirements.txt      # Python dependencies
│   └── mnist_cnn.pth         # Pretrained MNIST CNN weights
│
└── unity/
    └── cnn_visualizer/        # Unity XR project (URP + XR Interaction Toolkit / OpenXR)
        └── Assets/
            ├── Scripts/
            │   ├── CNNWebSocketCLient.cs   # Connects to the Python server
            │   ├── WSManager.cs            # WebSocket session/message management
            │   ├── DataModels.cs           # C# models mirroring the server's JSON payload
            │   ├── CNNBlockAnimator.cs     # Animates conv block reveals
            │   ├── FCNetworkVisualizer.cs  # Renders the fully-connected layer graph
            │   ├── ImageCubeSpawner.cs     # Spawns feature-map cubes in 3D space
            │   ├── WhiteboardDrawer.cs / whiteboardexporter.cs  # In-VR drawing → PNG export
            │   └── ...                    # Additional interaction/utility scripts
            └── Samples/                    # XR Interaction Toolkit / OpenXR / XR Hands sample assets
```

## Requirements

- Python 3.9+
- Unity (URP project using the XR Interaction Toolkit, OpenXR, and XR Hands packages) with a compatible XR headset or the XR Device Simulator
- Python packages (see `server/requirements.txt`):
  - `torch`, `torchvision`, `websockets`, `numpy`, `Pillow`

## Getting started

1. **Install Python dependencies**
   ```bash
   cd server
   pip install -r requirements.txt
   ```
2. **Run the WebSocket server**
   ```bash
   python server.py
   ```
   The server listens on `ws://0.0.0.0:8765` by default (override with the `PORT` environment variable), loads `mnist_cnn.pth`, watches `unity/cnn_visualizer/Assets/Pics/` for the latest exported PNG, and streams activations every 10 seconds.
3. **Open the Unity project**
   Open `unity/cnn_visualizer` in the Unity Editor, set `CNNWebSocketClient`'s `serverUrl` if needed, and press Play (or deploy to a headset). Draw a digit on the in-VR whiteboard — it will be exported as a PNG, picked up by the server, and the resulting activations will be visualized in the scene.

## Roadmap / open tasks

From the project's internal notes:
- Add a dynamic activation function toggle on the current model (ReLU, sigmoid, tanh)
- Restructure the network from 2 to 5 layers
- (Optional) add masking for weights
- Visualize training metrics: loss vs. epoch, accuracy vs. epoch, confusion matrix
- (Optional) Grad-CAM style explainability (XAI)

## ⚠️ Security note

While reviewing this repo, a **live-looking GitHub personal access token** was found committed in plain text inside `tasks.md`. Please revoke/regenerate that token in your GitHub settings (Settings → Developer settings → Personal access tokens) immediately and remove it from the file/history — treat it as compromised since it's been pushed to a public repo.

Separately, the Unity project currently has build/IDE artifacts checked into git (e.g. `Library/`, `.vs/`, `il2cppOutput/`, `.plastic/`), which bloats the repo significantly. A `.gitignore` cleanup (removing these from tracking) is recommended as part of the broader repo cleanup you mentioned.
