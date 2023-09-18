# Flutter Development Environment Dockerfile

This repository contains a Dockerfile for generating a development environment for Flutter. This allows developers to quickly get started with Flutter development without worrying about setting up their development environment from scratch.

The Docker image is designed to be used in Cloud/Self-hosted Development Environments like [DevPod](https://devpod.sh/), eliminating the need to install the Flutter and Android SDK on your local machine. The image includes an emulator device (Pixel with Android 33) and can be used with a shell alias to make it indistinguishable from a local installation. If you are using VSCode, you can also use this image as your [devcontainer](https://containers.dev/).

## Key Features

* **Cloud and Self-hosted Ready**: This Docker image is optimized for seamless integration with cloud or self-hosted development environments like DevPod. It eliminates the need to burden your local machine with Flutter and Android SDK installations.

* **Emulated Device**: Our Docker image comes pre-configured with an emulator device (Pixel with Android 33). This inclusion simplifies the testing and development process by providing a ready-to-use testing environment.

* **Shell Alias Integration**: For an experience virtually indistinguishable from a local installation, set up a shell alias with this Docker image. This enables you to work with Flutter effortlessly and efficiently.

* **VSCode Devcontainer Compatible**: If you prefer using Visual Studio Code (VSCode), you can seamlessly employ this Docker image as your devcontainer. It's tailored to simplify your development workflow within your favorite code editor.

## Customization

Feel free to tailor the Dockerfile to match your specific development requirements. You can add extra dependencies, configure environment variables, or select a different Flutter version to suit your project's needs.

## Getting Started

### Entrypoints

* `flutter` (default)
* `flutter-android-emulator`
* `flutter-web`

### Dependencies

- Make sure `KVM` is properly set up on your host ([instructions](https://developer.android.com/studio/run/emulator-acceleration#hypervisors))

When you want to run the `flutter-android-emulator` entrypoint your host must support KVM and have `xhost` installed.

#### flutter (default)

Executing e.g. `flutter help` in the current directory (appended arguments are passed to flutter in the container):

```shell
docker run --rm -e UID=$(id -u) -e GID=$(id -g) --workdir /project -v "$PWD":/project dhassault/flutter-dev help
```

When you don't set the `UID` and `GID` the files will be owned by `G-/UID=1000`.

#### flutter (connected usb device)

Connecting to a device connected via usb is possible via:

```shell
docker run --rm -e UID=$(id -u) -e GID=$(id -g) --workdir /project -v "$PWD":/project --device=/dev/bus -v /dev/bus/usb:/dev/bus/usb dhassault/flutter-dev devices
```

#### flutter-android-emulator

To achieve the best performance we will mount the X11 directory, DRI and KVM device of the host to get full hardware acceleration:

```shell
xhost local:$USER && docker run --rm -ti -e UID=$(id -u) -e GID=$(id -g) -p 42000:42000 --workdir /project --device /dev/kvm --device /dev/dri:/dev/dri -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY -v "$PWD":/project --entrypoint flutter-android-emulator  dhassault/flutter-dev
```

#### flutter-web

You app will be served on localhost:8090:

```shell
docker run --rm -ti -e UID=$(id -u) -e GID=$(id -g) -p 42000:42000 -p 8090:8090  --workdir /project -v "$PWD":/project --entrypoint flutter-web dhassault/flutter-dev
```

## VSCode devcontainer

You can also use this image to develop inside a devcontainer in VSCode and launch the android emulator or web-server. The android emulator need hardware acceleration, so their is no best practice for all common operating systems.

### Linux #1 (X11 & KVM forwarding)

For developers using Linux as their OS I recommend this approach, because it's the overall cleanest way.

Add this `.devcontainer/devcontainer.json` to your VSCode project:

```json
{
    "name": "Flutter",
    "image": "dhassault/flutter-dev:latest",
    "customizations": {
        "vscode": {
            "extensions": [
                "dart-code.dart-code",
                "dart-code.flutter",
            ]
        }
    },
    "runArgs": [
        "--device",
        "/dev/kvm",
        "--device",
        "/dev/dri:/dev/dri",
        "-v",
        "/tmp/.X11-unix:/tmp/.X11-unix",
        "-e",
        "DISPLAY"
    ],
    "postCreateCommand": "sudo chmod 777 /dev/kvm"
}
```

When VSCode has launched your container you have to execute `flutter emulators --launch flutter_emulator` to startup the emulator device. Afterwards you can choose it to debug your flutter code.

### Linux #2, Windows & MacOS (using host emulator)

Add this `.devcontainer/devcontainer.json` to your VSCode project:

```json
{
  "name": "Flutter",
  "image": "dhassault/flutter-dev:latest",
  "customizations": {
        "vscode": {
            "extensions": [
                "dart-code.dart-code",
                "dart-code.flutter",
            ]
        }
    },
}
```

Start your local android emulator. Afterwards reconnect execute the following command to make it accessable via network:

```shell
adb tcpip 5555
```

In your docker container connect to device:

```shell
adb connect host.docker.internal:5555
```

You can now choose the device to start debugging.

## Contributing

If you're interested in contributing to this project, PRs are welcome.

## License

This project operates under the MIT License - see the LICENSE file for full details.

## Acknowledgments

We extend our gratitude to the Flutter and Docker communities for their invaluable contributions.
