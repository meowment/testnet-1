<strong><p style="font-size:14px" align="left">Founder :
<a href="https://discord.gg/JqQNcwff2e" target="_blank">NodeX Capital Discord</a></p></strong>
<strong><p style="font-size:14px" align="left">Visit Our Website : 
<a href="https://nodex.codes/" target="_blank">https://nodex.codes</a></p></strong>
<strong><p style="font-size:14px" align="left">Follow Me :
<a href="https://twitter.com/nodexploit/" target="_blank">NodeX Twitter</a></p></strong>
<hr>

<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/50621007/183283696-d1c4192b-f594-45bb-b589-15a5e57a795c.png">
</p>

# Chain upgrade to commit 4ec1b0ca818561cef04f8e6df84069b14399590e
## (OPTION 1) Manual upgrade
Once the chain reaches the upgrade height, you will encounter the following panic error message:\
`ERR UPGRADE "xxx" NEEDED at height: 155420`
```
sudo systemctl stop strided
cd $HOME && rm -rf stride
git clone https://github.com/Stride-Labs/stride.git && cd stride
git checkout 4ec1b0ca818561cef04f8e6df84069b14399590e
make build
sudo mv build/strided $(which strided)
sudo systemctl restart strided && journalctl -fu strided -o cat
```

!!! DO NOT UPGRADE BEFORE CHAIN RECHES THE BLOCK `155420`!!!

### (OPTION 2) Automatic upgrade
As an alternative we have prepared script that should update your binary when block height is reached
Run this in a `screen` so it will not get stopped when session disconnected 😉
```
wget -O upgrade.sh https://raw.githubusercontent.com/nodesxploit/testnet/main/stride/upgrade/155420/upgrade.sh && chmod +x upgrade.sh && ./upgrade.sh
```
