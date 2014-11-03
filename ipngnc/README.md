ipngnc 
====================================

For the most, this is the standard Python NCClient developed by Shikar Bhushan and now maintained by Leonidas Poulopoulos. 

The NCClient library is available under the standard Apache 2 License.

So what have I modified/added?

A new method called 'send_command' which allows you to craft your own NETCONF envelope to be sent.

The reason for this modification was down to differing NETCONF versions amongst Cisco hardware initially.

The new method will still wrap your XML with the XML header and XML MSG_DELIM = "]]>]]>".

The issue when using namespaces is the xmlns declarations appear in the root element and not the nf: namespaced element.
Therefore, it should be relatively easy using an lxml tree to create test data with an nf:rpc root element and nxos: child elements
and post the data to the 'send_command' ncclient function.

How do I use this?

```
from ipngnc import manager

from lxml import etree
      

device = "x.x.x.x"
credentials = {'username' : 'x', 'password':'x'}

test_data = """<nf:rpc xmlns:nf="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:vlan_mgr_cli="http://www.cisco.com/nxos:1.0:vlan_mgr_cli" 
xmlns:nfcli="http://www.cisco.com/nxos:1.0:nfcli" xmlns:nxos="http://www.cisco.com/nxos:1.0" 
	xmlns:if="http://www.cisco.com/nxos:1.0:if_manager" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="%s">

<nxos:exec-command>
<nxos:cmd>%s</nxos:cmd>
</nxos:exec-command>
</nf:rpc>"""

send_string = test_data % ("%s", "clear ip arp 1.1.1.1 vrf management")
# NOTE-> the first %s is for the message ID. Always leave the first replacement alone!


def nxos_connect(host, port, user, password):
    return manager.connect(host=host, port=port, username=user, 
                         password=password, hostkey_verify=False, device_params={'name':'nexus'})


def nxos_cmd_test(command_):

    result = ""
    
    with nxos_connect("192.168.59.4", "22", 'admin','cisco') as m:
        print "Connected...\n"
        result = m.send_command(command_)
    return result
    
 if __name__ == '__main__':
 	root =  nxos_cmd_test(test_data)
```