# zerotier-prototype

Adding support for automatic management of ZeroTier Tunnels.
#### `Note:` The prototype code for the following repositories can be found in my `zerotier-proto` branch of my fork repository.

**For eg**

```bash
git clone git@github.com:Aryamanz29/<OPENWISP_REPO_NAME_HERE>.git
git checkout zerotier-proto
```

![-------------------------------------------------------------](https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png)




### `netjsonconfig`

- #### **Prototype Implementations:**
  
  - [x] Added support for generating ZeroTier configuration for OpenWrt devices.
  
  - [x] Added a ZeroTier backend that generates network configuration accepted by REST API endpoints of the ZeroTier Controller.

- #### **Prototype Code:** 
  - https://github.com/Aryamanz29/netjsonconfig/tree/zerotier-proto
  - **Unittests**
    - Zerotier backend: https://github.com/Aryamanz29/netjsonconfig/tree/zerotier-proto/tests/zerotier
    - OpenWRT backend: https://github.com/Aryamanz29/netjsonconfig/blob/zerotier-proto/tests/openwrt/test_zerotier.py  

#### **Prototype Demo:**

```py
# test_zerotier.py
from netjsonconfig.backends.openwrt.openwrt import OpenWrt
from netjsonconfig.backends.zerotier.zerotier import Zerotier

test_netjson = {
    "zerotier": [
        {
            "name": "network-network-1",
            "enableBroadcast": False,
            "id": "79vgjhks7ae448c5",
        },
        {
            "name": "zerotier-network-2",
            "enableBroadcast": True,
            "id": "yt6c2e21c0fhhtyu",
        },
    ]
}

test_uci = """package zerotier

config zerotier 'network_network_1'
        option enabled '0'
        list join '79vgjhks7ae448c5'

config zerotier 'zerotier_network_2'
        option enabled '1'
        list join 'yt6c2e21c0fhhtyu'
"""

o1 = Zerotier(test_netjson).render()

print('---- Zerotier Backend : Network Configuration -----\n')
print(o1, end='\n')


client_config = Zerotier.auto_client(
    host='my.zerotier.com', server=test_netjson['zerotier'][0]
)

o3 = OpenWrt(test_netjson).render()

print('---- OpenWRT Backend : Zerotier Configuration (NetJSON -> UCI) -----\n')
print(o3, end='\n')

o4 = OpenWrt(native=test_uci).config
# for pretty print ordered dictionary
from json import dumps
print('---- OpenWRT Backend : Zerotier Configuration (UCI -> NetJSON) -----\n')
print(dumps(o4, indent=4))

```

```bash
(env) ➜  tests git:(zerotier-proto) ✗ py -m unittest test_zerotier.py
---- Zerotier Backend : Network Configuration -----

# zerotier config: 79vgjhks7ae448c5

enableBroadcast=False
n=network-network-1
nwid=79vgjhks7ae448c5

# zerotier config: yt6c2e21c0fhhtyu

enableBroadcast=True
n=zerotier-network-2
nwid=yt6c2e21c0fhhtyu


---- OpenWRT Backend : Zerotier Configuration (NetJSON -> UCI) -----

package zerotier

config zerotier 'network_network_1'
        option enabled '0'
        list join '79vgjhks7ae448c5'

config zerotier 'zerotier_network_2'
        option enabled '1'
        list join 'yt6c2e21c0fhhtyu'

---- OpenWRT Backend : Zerotier Configuration (UCI -> NetJSON) -----

{
    "zerotier": [
        {
            "id": "79vgjhks7ae448c5",
            "name": "network-network-1",
            "enableBroadcast": false
        },
        {
            "id": "yt6c2e21c0fhhtyu",
            "name": "zerotier-network-2",
            "enableBroadcast": true
        }
    ]
}

----------------------------------------------------------------------
Ran 0 tests in 0.000s

OK
```

### `openwisp-controller`

- #### **Prototype Implementations:**

  - [x] Added a new Zerotier VPN backend.
  - [x] Created a Zerotier network using the `ZerotierCentralAPI` when adding a new VPN server.
  - [x] Added automatic generation of templates for the Zerotier VPN backend, similar to the OpenVPN and WireGuard VPN backends.
  - [x] Applied the Zerotier `VPN-client` template to the OpenWRT device for generating Zerotier configuration.
  - [x] Deletion of a Zerotier VPN server through OpenWISP will also remove it from Zerotier Central (ie. `https://my.zerotier.com/`).

- #### **Prototype Code:** https://github.com/Aryamanz29/openwisp-controller/tree/zerotier-proto

#### **Prototype Demo:**

https://user-images.githubusercontent.com/56113566/229103195-8e284223-7521-4778-bd0e-1f10e5c80193.mp4


### `netdiff` and `openwisp-netowork-topoloy`

- #### **Prototype Implementations:**

  - [x] Created `ZerotierParser` in `netdiff` that can parse the Zerotier peer information.
  - [x] Integrated netdiff `ZerotierParser` with OpenWISP Network Topology.
  - #### **Prototype Code:**
   
    - https://github.com/Aryamanz29/netdiff/tree/zerotier-proto
    
    - https://github.com/Aryamanz29/openwisp-network-topology/tree/zerotier-proto

#### **Prototype Demo:**

https://user-images.githubusercontent.com/56113566/229118545-055bb360-8eac-49e2-9b3a-0b7003426154.mp4

![-------------------------------------------------------------](https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png)