# Shader SDK (WIN)

In this tutorial we wil show how to setup Beam Shader SDK on Windows OS

{% hint style="warning" %}
The Shader SDK's current version can be characterized as an 'early developer preview', with a high possibility of encountering unusual issues and bugs.
{% endhint %}

### Prerequisites

You will need Microsoft Visual Studio, community edition will do just fine. In this tutorial I have used the 2022 version.&#x20;

You will also need a Python 3.x installed and available in the path (check by typing 'python' in the command line)

### Building Shader SDK

1. Clone Shader SDK repository from here: [https://github.com/BeamMW/shader-sdk](https://github.com/BeamMW/shader-sdk)
2. Open Developer Command Prompt for VS. This can be done from either Tools Menu within the VS IDE or by finding an entry in the Start menu items
3. Change directory to the location of the shader-sdk and run: `shade init`

This will operation will install and build all the necessary components.

<figure><img src=".gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

### Creating a new Shader Project

1. Open a Developer Console and create a new folder for your Shader project. We will use 'vault' in our example

<figure><img src=".gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

2\. Set the following environment variables to point to the location of you shader-sdk and wasi-sdk:

`set BEAM_SHADER_SDK=c:\Code\shader-sdk`

`set WASI_SDK_PREFIX=C:\Code\shader-sdk\wasi-sdk-14.0`

3\. Run the following command: `shade create_project <project name>`

In our case:

`c:\Code\shader-sdk\shade create_project vault`

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

This will initialize the structure of the Shader project, which should look as follows:

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>



### Building the Shader Project

In the Visual Studio, chose '**Open A Local Folder**' from the starter screen

<figure><img src=".gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

Navigate to the project folder and clock 'Select Folder'. This will generate a CMake project. The end result should be visible in the Solution Explorer window and look like this:

<figure><img src=".gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Choose the wasm32-Release build configuration

<figure><img src=".gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

This will build the .wasm files which will be located in the following folder:

`C:\Code\vault\build\wasi\shaders`



{% hint style="warning" %}
In case something goes wrong with this step. Try the following before calling support:

1. Delete the 'out' folder using Windows Explorer
2. In the 'Project' menu, choose 'Delete Cache and Reconfigure'
3. Rebuild the 'wasm32-Release'&#x20;
{% endhint %}

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>



Now you are ready to start coding your Shader!
