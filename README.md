# Simple Industrial Networking ChatOps Example with NETCONF and Cisco Webex

This is a simple ChatOps example in the industrial networking space. It will show how easy it is to implement ChatOps with one simple script!

## Challenge
It can happen that Operational Technology (OT) operators need to deal with complicated (network) configuration interfaces, unnecessary delays because of unplanned configuration changes and many other known IT/OT convergence challenges.

## Solution

By introducing **Industrial NetDevOps** and specifically **Industrial ChatOps** into the industrial network, the life of the OT worker will get easier.

An OT worker can now easily change the network configuration and many other application settings via simple text messages on Cisco Webex Teams. What changes the OT worker is allowed to make will be pre-defined by the network & application operations team which will also lower their workload by automating these requests.

This **example screenshot** shows how easy it is to enable/disable an interface on an IR1101:

![](images/webex-bot.png)

### Simple Script to change one Interface on the IR1101

**Run a Webhook (Webex Teams)**: After registering a Webex Teams bot on [developer.webex.com](https://developer.webex.com), we start running our Webex Teams bot on port 5000 and define the function `change_interface4` to call when the OT worker is requesting the change via the bot `/changeint4`.

```
# Add new commands to the box.
bot.add_command("/getinfo", "Get basic device information (hostname, IOS version)", getinfo)
bot.add_command("/getallinterfaces", "Get information from all interfaces", getallinterfaces)
bot.add_command("/changeint4", "disable or enable Interface 4: e.g. /changeint4 disable", change_interface4)
bot.remove_command("/echo")
bot.set_help_message("Welcome to the IoT Config Bot! You can use the following commands:\n")

# Run Bot
bot.run(host="0.0.0.0", port=5000)
```

**NETCONF Function to change the IR1101 Interface FastEthernet0/0/4**: After the function is called, the user input will be checked. If the user wrote `/changeint4 enable` then the interface will be enabled `<enabled>true</enabled>`. If the user wrote `/changeint4 disabled` then the interface will be disabled `<enabled>false</enabled>`. As we covered in the NETCONF learning lab, the IOS configuration in XML will be applied on the IR1101.

```
# Enable or disable the interface of FastEthernet 4
# depends on the user input
def change_interface4(incoming_msg):
    config = '''
        <config xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
              <interfaces xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces">
                <interface>
                    <name>FastEthernet0/0/4</name>
                    <enabled>true</enabled>
                </interface>
              </interfaces>
          </config>
        '''
    config_dict = xmltodict.parse(config)

    # most basic differentiation of the user's command.
    if incoming_msg.text == "/changeint4 enable":
        netconf_reply = m.edit_config(target='running', config=config)
        return "Int4 was successfully enabled."
    elif incoming_msg.text == "/changeint4 disable":
        config_dict["config"]["interfaces"]["interface"]["enabled"] = "false"
        config = xmltodict.unparse(config_dict)
        netconf_reply = m.edit_config(target='running', config=config)
        return "Int4 was successfully disabled."
    else:
        return "Wrong command, please try again."
```

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Further Links

* [Cisco DevNet Website](https://developer.cisco.com)
* [Learning Labs: Industrial NetDevOps - Getting Started](https://developer.cisco.com/learning/modules/industrial-netdevops)
* [IR1101 DevNet Sandbox](https://devnetsandbox.cisco.com/RM/Diagram/Index/a2046279-a193-4d22-87b3-abcfee9569a6?diagramType=Topology)