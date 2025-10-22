## Using DevTunnels

We will now [setup a DevTunnel](https://learn.microsoft.com/en-us/azure/developer/dev-tunnels/get-started?tabs=windows) to allow you to call your extension from the web. 

1. Open a new terminal windows
2. Issue a `devtunnel login` command.
    1. Select "Work or School Account"
    2. Your login credentials can be found in the resources tab above the instructions.
5. Issue a `devtunnel host -p 5181 --allow-anonymous` command
6. Copy the URL that is output in the terminal.

Your locally running endpoint is now accessible via a hosted DevTunnel to the internet.