# Contrail DHCP
When 'enable-dhcp', the DHCP requests from the VMs are trapped to contrail-vrouter-agent, 
which responds with the IP address of the VM's interface along with other configured DHCP options.





      +-----------------------------------------+
      |               DHCP service              |
      |                 v      ^                |
      |SendDhcpResponse v      ^ HandleVmRequest|
      |-----+           v      ^                |
      |Agent|           v      ^                |
      +------------------ pkt0------------------+
                           |
                           |
                           |
      +-------------------vif2------------------+
      |       agent_rx  v       ^  vr_trap      |
      |                 v       ^               |
      |-------+         v       ^               |
      |Vrouter| vif_tx  v       ^  bridge_input |
      +------------------vifXXX-----------------+
                           |
                           |
                           |
      +------------------tapXXX-----------------+
      |                 v       ^               |
      |    DhcpResponse v       ^ DhcpRequest   |
      |--+              v       ^               |
      |VM|                                      |
      +-----------------------------------------+
