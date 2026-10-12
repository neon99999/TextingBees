# TouchDesigner + Raspberry Pi SMS/OSC Connection

## 1. Log into the Pi

Connect to the Raspberry Pi over SSH using PuTTY.

Use the Pi hostname/IP and the username/password written on the box.

Typical username:

```text
pi
```

Once connected, you should see a prompt like:

```text
pi@raspberrypi:~ $
```

## 2. Go to the TouchDesigner project

```bash
cd /home/pi/TouchDesignerBeeSign
```

## 3. Check the TouchDesigner IP and OSC port

```bash
grep -E 'TOUCHDESIGNER_IP|TOUCHDESIGNER_PORT|TOUCHDESIGNER_CONFIRM_PORT' .env
```

Expected format:

```text
TOUCHDESIGNER_IP=192.168.1.76
TOUCHDESIGNER_PORT=7006
TOUCHDESIGNER_CONFIRM_PORT=XXXX
```

The `TOUCHDESIGNER_IP` should be the IP address of the computer running TouchDesigner.

## 4. Change the TouchDesigner IP

```bash
nano .env
```

Change:

```text
TOUCHDESIGNER_IP=192.168.1.76
```

Save with:

```text
Ctrl+O
Enter
Ctrl+X
```

Then restart the sender:

```bash
sudo supervisorctl restart touchdesigner-touchdesigner
```

This restart is required for the new IP to take effect.

## 5. Check that everything is running

```bash
sudo supervisorctl status
```

Relevant services should show `RUNNING`, especially:

```text
touchdesigner-sms
touchdesigner-touchdesigner
touchdesigner-web
```

## 6. Check Twilio messages

To confirm the Pi is receiving SMS messages from Twilio:

```bash
sudo supervisorctl tail -f touchdesigner-sms
```

Press `Ctrl+C` to exit.

You should see Twilio requests returning `200` and received messages.

## 7. Check messages being sent to TouchDesigner

```bash
sudo supervisorctl tail -f touchdesigner-touchdesigner
```

You should see:

```text
Sending to TouchDesigner: ...
Sent composite_id=...
```

If TouchDesigner replies correctly, you should also see:

```text
TD confirmed message ...
```

If you see:

```text
No confirmation from TD ...
```

then TD is not sending the acknowledgment back to the Pi.

## 8. TouchDesigner OSC setup

TouchDesigner listens on:

```text
Port: 7006
Address: /sms/message
```

The Pi sends JSON containing fields such as:

```text
id
body
phone_number
received_at
```

TouchDesigner sends confirmation back using:

```text
/sms/confirm
```

The OSC Out DAT in TouchDesigner must point to the Raspberry Pi IP and use the same confirmation port configured as:

```text
TOUCHDESIGNER_CONFIRM_PORT
```

## 9. Basic network test

From the Pi:

```bash
ping 192.168.1.76
```

Replace the IP with the TouchDesigner computer's IP.

If the ping works, the Pi can reach the TD machine on the network.

## Most common fix

If the TouchDesigner computer's IP changes:

```bash
cd /home/pi/TouchDesignerBeeSign
nano .env
```

Update the IP, save, then run:

```bash
sudo supervisorctl restart touchdesigner-touchdesigner
```

That was the issue in this case.