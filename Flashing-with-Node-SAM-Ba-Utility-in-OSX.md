### Node-Sam-Ba
This is a nodejs utility that you can flash your ARM based g2 boards with.

## Installing NodeJs
The easiest way to install NodeJs on OSX is to use the [node version manager](https://github.com/creationix/nvm).  Use this [one-liner](https://github.com/creationix/nvm#install-script) at the command line to install nvm:<br>
`curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.4/install.sh | bash`
<p>
After nvm is installed type:\
`nvm install 6` <br>
This will install node version 6.

## Getting node-sam-ba
Next we use git to download sam-node-ba like the screen shot below:<br>
`git clone https://github.com/synthetos/node-sam-ba.git`
<br><br>
![](https://c1.staticflickr.com/5/4388/36654186324_b987109e4c_c.jpg)

## Installing node-sam-ba
Installing is pretty simple.  Change to the sam-node-ba directory and then issue the npm command to download and install all needed dependencies.<br>
**Type:**
`cd sam-node-ba`<br>
`npm install`<br><br>
![](https://c1.staticflickr.com/5/4392/36693403503_bb9baab96a_c.jpg)
## Usage
To use the node-sam-ba programming utility you need to make sure your USB cable is hooked to your g2core board and that the board is in bootloader mode. *IE: No heart beats blinking.* See entering bootloader mode for more details on this.<br>
You can now type: `node flash.js` to get the command usage.

![](https://c1.staticflickr.com/5/4499/23511638068_281b120bf1_c.jpg)
<p>
You will notice that you must provide a port (-p option) to specify which serial port the g2core board is identified in your computer.  If you do not see a port make sure that your board is hooked up and is in bootloader mode.

<p>
My command looked something like this: 
node flash.js -p /dev/tty.usbmodem146231 -b ~/Documents/GitHub/g2/g2core/bin/Ultimaker2Plus-gquintic-c/g2core.bin

<p>
INCOMPLETE: Attach Video and screen shot. -ril3y