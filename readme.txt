*** No warranty for this software. ***
*** Use at own risk. ***

This is a repo to manage Sibcoin masternodes using ansible.
This assumes you have ansible installed on a mac or linux computer.
(Ansible does not work well running from a Windows host.)

To get Ansible installed on an Ubuntu instance:
apt-get update
apt-get install ansible

The detailed steps are for one master node, but you could manage multiple masternode remotes.

To learn more about the steps in creating a Sibcoin masternode see:
https://github.com/ivansib/sibcoin/blob/master/doc/guide-startmany.md

For sentinel, see https://github.com/ivansib/sentinel (Note: use sibcoin branch!)

Desktop wallet: (This is probably a windows or mac.) 
1. Start Sibcoin-Qt.
2. Wait for it to completely sync.
3. Encrypt the local wallet by opening "Settings", "Encrypt wallet". Note: The wallet will close.
4. Re-start Sibcoin-Qt.
5. Ensure you can unlock the wallet by using the password that you used in step 3 by going
   to "Settings" and "Unlock wallet". If you do not remember the password, do not use this 
   wallet! (Note: The little lock in the bottom right corner of the wallet will show text
   when you hover over it. It should say "Wallet is encrypted and currently unlocked."
6. Click on "receive" tab.
7. Click on "request payment" button.
8. Copy the receiving address. It will look something like this: SQiQEhFSDK5xQyi6CUTGEaRBFEhkoqSRa2
9. Send exactly 1000 Sibcoin to the receiving address in the receiving address above.
10. Open that transaction where the 1000 Sibcoin was received and copy the 
   transaction id . It will look something like 
   this: 353602de3a0150f4d1251aa9bf794fc626f3bbda921fc9a6014ff76e6c056f07
   Note: You do not need the trailing "-000" value.
   Copy that transaction id and paste it into https://chain.sibcoin.net/en.
   Find out if the output is a "0" or a "1". In the case of the example transaction,
   the value is a "1".
11. Open up debug window by going into "Tools", "Debug Console".
12. Type in the following command: "masternode genkey" (the output is the masternodeprivkey)
   It will look something like this: 5J***********************************************53
   Note: I changed most of the values to '*' to obscure the key.
13. You will need to wait for 15 confirmations of the 1000 Sibcoin before you can continue.
   This is fine because we need to go setup the remote host now.

Setup the remote hosts: 
# Note: I strongly recommend setting up a pre-shared key. Click on the help and follow the 
# directions.
14. Create a new linux host on someone like Digital Ocean. 
   (ex: Create droplet, $5/month, Ubuntu 16, any datacenter, add preshared key, give it a hostname like
   mn1 or something)
15. Add the remote host to the hosts file in this directory.
16. Edit the ansible.cfg file for the location of your private file. 
17. Run the entire playbook on the hosts (this will also install the necessary stuff for ansible to run)
  ansible-playbook -v sibcoin.yml 
    or if you just want to run it on a few hosts, use the '-l' for "limiting which hosts:
  ansible-playbook -v sibcoin.yml -l mn2,mn4
18. Ensure you can connect to all of the hosts in the hosts file.
   ansible -m shell -a 'hostname' all
19. Connect to the remote host. 
20. See if the sibcoin process is started by running "ps -ef | grep sibcoin"

The output should look something like this:
root     18428     1 13 Jul17 ?        03:11:44 /sibcoin/sibcoin-0.16.1/bin/sibcoind
root     22222 22183  0 03:06 pts/0    00:00:00 grep --color=auto sibcoin

    You can also check the status of the block chain syncing 
    by running: "/sibcoin/sibcoin-0.16.1/bin/sibcoin-cli mnsync status"

Here's an sample output:
#   /sibcoin/sibcoin-0.16.1/bin/sibcoin-cli mnsync status
{
  "AssetID": 999,
  "AssetName": "MASTERNODE_SYNC_FINISHED",
  "Attempt": 0,
  "IsBlockchainSynced": true,
  "IsMasternodeListSynced": true,
  "IsWinnersListSynced": true,
  "IsSynced": true,
  "IsFailed": false
}

  While we wait for the Sibcoin block chain to sync, we can do a few other things.

21. Run "cd .sibcoin"
22. Edit the sibcoin.conf file: 
 a. Change the rpcuser (the 'xxx' value below) and the the rpcpassword (the 'yyy' value below)
    to random values, but probably best to only use alphanumeric values.
 b. Change the masternodeprivkey value to be the value used in prior step above.

sibcoin.conf: (on remote)
  #----
  rpcuser=xxx
  rpcpassword=yyy
  rpcallowip=127.0.0.1
  #----
  listen=1
  server=1
  daemon=1
  maxconnections=256
  logtimestamps=1
  #--------------------
  masternode=1
  masternodeprivkey=5J***********************************************53

Note: There is no need for a msternode.conf on the remote.

22. Back on the Desktop, change into the directory that has the masternode.conf file:
    cd ~/Library/Application\ Support/Sibcoin/   (if mac)
23. Make the masternode.conf file look something like this: (but change the ip address, t

masternode.conf (on Desktop/local):
   # alias ip:1945 masternodeprivkey transaction_of_1000_sib output_index(0or1)
   mn1 123.123.123.123:1945 5J***********************************************53 353602de3a0150f4d1251aa9bf794fc626f3bbda921fc9a6014ff76e6c056f07 1

24. Restart the Desktop/local wallet.
25. Once the remote server has downloaded the entire blockchain, you can continue.
26. On Desktop, go into the "Masternodes" tab, click on the "mn1" line, and click start-alias.
    You will be prompted for the password that was used to encrypt the wallet.
    The line should change to "PRE_ENABLED".
27. You can go into "debug console" and run these commands: (informational only)

  masternode list-conf

  {
  "masternode": {
    "alias": "mn1",
    "address": "123.123.123.123:1945",
    "privateKey": "5J***********************************************53",
    "txHash": "353602de3a0150f4d1251aa9bf794fc626f3bbda921fc9a6014ff76e6c056f07",
    "outputIndex": "1",
    "status": "MISSING"
    }
  }

  masternode outputs

  {
    "353602de3a0150f4d1251aa9bf794fc626f3bbda921fc9a6014ff76e6c056f07": "1"
  }

27. Kill the remote wallet. On the remote, run: "pkill sibcoind"
Note: If you want to kill the running processes on all of your remotes:
  ansible -m shell -a 'pkill sibcoind' all
28. Restart the remote wallet by running "/sibcoin/sibcoin-0.16.1/bin/sibcoind"
29. Check the status by running "/sibcoin/sibcoin-0.16.1/bin/sibcoin-cli mnsync status"

This output should eventually get to:
{
  "AssetID": 999,
  "AssetName": "MASTERNODE_SYNC_FINISHED",
  "Attempt": 0,
  "IsBlockchainSynced": true,
  "IsMasternodeListSynced": true,
  "IsWinnersListSynced": true,
  "IsSynced": true,
  "IsFailed": false
}

30. When that is done, then run this: "/sibcoin/sibcoin-0.16.1/bin/sibcoin-cli masternode status"

{
  "vin": "CTxIn(COutPoint(0182546******************************************************027, 0), scriptSig=)",
  "service": "123.123.123.123:1945",
  "payee": "SQiQEhFSDK5xQyi6CUTGEaRBFEhkoqSRa2",
  "status": "Masternode successfully started"
}

31. The wallet should eventually change from "PRE_ENABLED" to "ENABLED".

   If it goes to "EXPIRED", you will need to stop both the Desktop wallet and the remote process, 
   do the "start-alias" again, then start the remote host. 
   (it means that it took too long for the remote to start after issuing the 'start-alias' command) 

Other commands/tips:
- If you want to run a command (like "sibcoin-cli mnsyc status"), you could run it like this:
ansible -m shell -a '/sibcoin/sibcoin-0.16.1/bin/sibcoin-cli mnsync status' mn1,mn3
  (where mn1 is an entry in the hosts file in this directory)

If you want to check the status of your masternode, visit:
https://sibcoin.org/stats or http://sibcoin.org/en/stats
