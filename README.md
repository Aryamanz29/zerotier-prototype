# zerotier-prototype


![gsoc23](https://user-images.githubusercontent.com/56113566/229188871-3526101e-9382-4da0-8142-f3e549ebbfd7.png)

<h3><p style="text-align: center;"><strong><i>Adding support for automatic management of ZeroTier Tunnels.</i></strong></p></h3>


#### **Notes:** 

- If you encounter any issues while playing the demo video files in this `README`, please consider switching to a chromium-based browser to play them without any errors.

- The prototype code for the following repositories can be found in my `zerotier-prototype` branch of my fork repository.

- **For eg**
  ```bash
  git clone git@github.com:Aryamanz29/<OPENWISP_REPO_NAME_HERE>.git
  git checkout zerotier-prototype
  ```

![-------------------------------------------------------------](https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png)




### `netjsonconfig`

- #### **Prototype Implementations:**
  
  - [x] Added support for generating Zerotier configuration for OpenWrt devices.
  
    **Steps:**

    - We can extend the Zerotier converter for OpenWrt backend in [openwrt/converters/zerotier.py](https://github.com/Aryamanz29/netjsonconfig/blob/zerotier-prototype/netjsonconfig/backends/openwrt/converters/zerotier.py)
  
    - The extended Zerotier converter configuration syntax logic is written as per [openwrt zerotier package](https://github.com/mwarning/zerotier-openwrt/wiki).

    - Added basic unit tests for the OpenWrt backend renderer and parsers. 
  
    - **Note:** Currently the [zerotier/schema.py](https://github.com/Aryamanz29/netjsonconfig/blob/zerotier-prototype/netjsonconfig/backends/zerotier/schema.py) only supports limited properties required for running this prototype, we can change this later on.


  - [x] Added a Zerotier backend that generates network configuration accepted by REST API endpoints of the ZeroTier Controller.

    **Steps:**

    - We can a schema for the [Zerotier backend](https://github.com/Aryamanz29/netjsonconfig/blob/zerotier-prototype/netjsonconfig/backends/zerotier/zerotier.py) that is based on Zerotier controller [OpenAPI specification](https://docs.zerotier.com/openapi/servicev1.json).
    
    - Now, according to the given schema, the `netjsonconfig.backends.zerotier` converter, parser, template, and renderer logic is written.
    
    - Added logic for automatic [generation of clients](https://github.com/Aryamanz29/netjsonconfig/blob/zerotier-prototype/netjsonconfig/backends/zerotier/zerotier.py#L16-L18) similar to **OpenVPN** and **Wireguard** in the Zerotier VPN backend that is later used by OpenWISP Controller config templates.
    
    - Added basic unit tests for the Zerotier backend renderer and parsers.

- #### **Prototype Code:** 
  
  - https://github.com/Aryamanz29/netjsonconfig/tree/zerotier-prototype
  
  - **Unittests**
        
    - OpenWRT backend: https://github.com/Aryamanz29/netjsonconfig/blob/zerotier-prototype/tests/openwrt/test_zerotier.py
    
    - Zerotier backend: https://github.com/Aryamanz29/netjsonconfig/tree/zerotier-prototype/tests/zerotier

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
(env) ➜  tests git:(zerotier-prototype) ✗ py -m unittest test_zerotier.py
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
![-------------------------------------------------------------](https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png)

### `openwisp-controller`

#### **Prototype Demo:**


https://user-images.githubusercontent.com/56113566/229542857-dbc1886d-20f8-4794-abf0-7a6473710fd4.mp4


- #### **Prototype Implementations:**

  - [x] Added a new Zerotier VPN backend.
    
    **Steps:**

    - We can add a [new VPN backend](https://github.com/Aryamanz29/openwisp-controller/blob/zerotier-prototype/openwisp_controller/vpn_backends.py#L75-L81) by extending netjonconfig Zerotier backend.

    - VPN backend schema should be similar to what we defined in `netjsonconfig`.

  - [x] Created a Zerotier network using the [ZerotierCentralAPI](https://docs.zerotier.com/central/v1/) when adding a new VPN server.
  
    ![networks](https://user-images.githubusercontent.com/56113566/229290021-49e3be9d-4621-4818-8dd6-cd1afe5ceb74.png)

    **Steps:**

    - To accomplish this, we can create a new Python class or API view class in the [config/api/zerotier_central_api.py](https://github.com/Aryamanz29/openwisp-controller/blob/zerotier-prototype/openwisp_controller/config/api/zerotier_central_api.py) file that we can use to manage ZeroTier networks through OpenWISP.

    - This class will enable us to perform CRUD operations on networks, members, and other relevant information.

    - **Note:** During the prototype phase, I exclusively utilized the [ZerotierCentralAPI](https://docs.zerotier.com/central/v1/) However, if we aim to manage services such as peers and [self-hosted zerotier network controllers](https://docs.zerotier.com/self-hosting/network-controllers/) in a way similar to the **zerotier-cli** and the zerotier central management portal **(my.zerotier.com)** through OpenWISP, we may need to utilize both the [ZerotierOneServiceAPI](https://docs.zerotier.com/service/v1/) and the [ZerotierCentralAPI](https://docs.zerotier.com/central/v1/).

  - [x] Added automatic generation of templates for the Zerotier VPN backend, similar to the OpenVPN and WireGuard VPN backends.

    **Steps:**

    - We can utilise the code block presented below to invoke the backend `auto_client` method and obtain a configuration dictionary that can be used as a template.
    
      ```py
        # If backend is 'zerotier' than auto_client and update the config
        elif self._is_backend_type('zerotier') and template_backend_class:
             auto = backend.auto_client(
                  host=self.host,
                  server=self.config[config_dict_key][0],
                  **context_keys,
              )
      ```

  - [x] Applied the Zerotier `VPN-client` template to the OpenWRT device for generating Zerotier configuration.
    
    **Steps:**

    - At present, the prototype only handles public Zerotier networks. In order to allow for private networks, it is necessary to authorize each member node with Zerotier once it has joined the network. One potential solution is to use OpenWISP to create Zerotier client identities in advance using the [`zerotier-idtool`](https://github.com/zerotier/ZeroTierOne/blob/dev/doc/zerotier-idtool.1.md). Then, when applying the template to an OpenWRT device, device configuration variables (which include the Zerotier secret key) and file configuration can be utilized to add the contents of the Zerotier identities to the device.

    - After creating the Zerotier identities and extracting the `member_id` using the `identity.secret`, we can use the [ZerotierCentralAPI](https://github.com/Aryamanz29/openwisp-controller/blob/zerotier-prototype/openwisp_controller/config/api/zerotier_central_api.py) class to make a call to the endpoint `https://my.zerotier.com/api/v1/network/{network_id}/member/{member_id}` with a JSON payload of `{"config": {"authorized": true}}`. This will authorize the private member in the Zerotier network.

    - Using [zerotier-idtool](https://github.com/zerotier/ZeroTierOne/blob/dev/doc/zerotier-idtool.1.md) to generate zerotier identities.
      
      ```bash
      zerotier-idtool generate identity.secret identity.public
      ```

    - Configure the device configuration variables and file configuration to manage the Zerotier identities when adding the Zerotier VPN client on the device.
    
    ![zerotier-identity](https://user-images.githubusercontent.com/56113566/229160497-06a03c25-f955-4563-9078-12a7f4a402e1.png)

   - *Note: We can automate the above-mentioned steps programmatically after discussion*.

  - [x] Deletion of a Zerotier VPN server through OpenWISP will also remove it from Zerotier Central (ie. https://my.zerotier.com).

    **Steps:**

    - We can add a method for `post_delete` signal to facilitate the automatic deletion of zerotier VPN networks.
    
    ```py
        @classmethod
        def post_delete(cls, instance, **kwargs):
            """
            class method for ``post_delete`` signal
            for managing automatic deletion of vpn servers
            """
            if not instance._is_backend_type('zerotier'):
                return
            network_id = instance.config.get('zerotier')[0].get('id')
            ZerotierCentralAPI(instance.host, instance.auth_token).delete_network(network_id)
    ```

- #### **Prototype Code:** https://github.com/Aryamanz29/openwisp-controller/tree/zerotier-prototype

![-------------------------------------------------------------](https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png)

### `netdiff and openwisp-network-topoloy`

#### **Prototype Demo:**

https://user-images.githubusercontent.com/56113566/229118545-055bb360-8eac-49e2-9b3a-0b7003426154.mp4


- #### **Prototype Implementations:**

  - [x] Created `ZerotierParser` in `netdiff` that can parse the Zerotier peer information and can return networkx graph object.

    **Steps:**

    - We can obtain information about the peers of devices in a network by using either the command `zerotier-cli peers -j` or the [ZerotierCentralAPI](https://github.com/Aryamanz29/openwisp-controller/blob/zerotier-prototype/openwisp_controller/config/api/zerotier_central_api.py) client of the `openwisp-controller`, which consumes the [zerotier peers endpoints](https://docs.zerotier.com/service/v1/#operation/getPeers).
    
    - We can implement the [ZerotierParser](https://github.com/Aryamanz29/netdiff/blob/zerotier-prototype/netdiff/parsers/zerotier.py) by extending the **netdiff** `BaseParser`.
    
    - Currently, the parsing logic is very simple. For every Zerotier peer with any role, a node is added to the graph instance along with other Zerotier peer properties. It's worth noting that we may make changes to this approach in the future.

    - For every valid Zerotier peer, we can gather all possible paths and parse them to add graph edges.

    - To determine graph edge cost, we can use peer **latency**.
  
  - [x] Integrated netdiff `ZerotierParser` with OpenWISP Network Topology.
  
    ```py
        DEFAULT_PARSERS = [
        ...
        ...
        ('netdiff.ZerotierParser', 'Zerotier'),
    ]
    ```

  - #### **Prototype Code:**
   
    - https://github.com/Aryamanz29/netdiff/tree/zerotier-prototype
    
    - https://github.com/Aryamanz29/openwisp-network-topology/tree/zerotier-prototype

![-------------------------------------------------------------](https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png)