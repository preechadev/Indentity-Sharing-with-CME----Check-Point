# cme_identitysharing

Original post:
https://community.checkpoint.com/t5/Cloud-Network-Security/ID-Sharing-on-AutoScaling/m-p/97207#



ID Sharing on AutoScaling

Use case; since the VMSS setup on Azure allow to customers connect the On-Prem to cloud with Native VPN GW or Express Route and redirect the traffic to the VMSS members (Check Point Gateways), like the next diagram.

Transit vNET based on VMSSTransit vNET based on VMSS

 

Now let's work on the need to use Identities on the rules and since this comes from the On-Prem, the identities will be based on AD, there is a good way to achieve this, in Check Point exist the ID-Sharing feature, where an On-Prem Check Point Gateway with connectivity to the AD can grab the Login events on the AD and create the list based on AD groups and share this to another GW, so the other GW don't need to reach the AD.

ID Sharing feature explainedID Sharing feature explained

 

But this needs to be configured per GW basis, this in autoscaling scenarios need automation, the CME tool that handles the automation of the CG IaaS deployment also have the capability to customize based on scripts that can run on Management side or GW side.

So I created a bash script that allows to enable the identity sharing on each VMSS new deployed GW.

First we need to enable the capability to share identities on the On-Prem GW, this is done on the Gateway object under Identity Awareness menu.

this is done on the On-Prem previously configured AD-Query GWthis is done on the On-Prem previously configured AD-Query GW

 

Once we prepare this, we need to set up the CME based on CloudGuard Network for Azure VMSS R80.10 and Higher Administration Guide (This link was edited by Check Point on 19 Dec 2021)

Once all is done, we need to download the bash script and pass it to the management, the script is here Github, you can use curl_cli to do it.

 

curl_cli -k 'https://raw.githubusercontent.com/christiancastilloporras/Indentity-Sharing-with-CME----Check-Point/master/cme_identitysharing.bash' -O

 

and give it execution permissions.

Then on the CME need to add this file to the Management, make sure to put the correct path;

 

autoprov-cfg set management -cs 'path/to/script'

 

 

Then on the template need to call the Custom Parameter feature, this feature call the management script and pass params to the script, the script uses 5 parameters, the 1 and 2 are passed by the CME by default each time you call this feature ('add' or 'delete' as $1 and newly deployed GW Name as $2), then the script needs an action this should be "IDSHARING", needs the rulebase to install; this needs to be exactly the name of the Policy Package of the deployed Gateways and exactly the GW name where the VMSS will grab identities; all this will setup the next addition to the CME;

 

autoprov-cfg set template -tn "name-of-template" -cp 'IDSHARING rulebase gwname'

 

 

And now every newly deployed GW will have the identities and we can set up rules for the VMSS based on AD Groups.
