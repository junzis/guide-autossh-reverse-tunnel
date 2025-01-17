# Setting Up an Auto-reconnecting SSH Reverse Tunnel

This guide explains how to create a persistent SSH reverse tunnel that automatically reconnects after disconnection.

## Use Case

This solution is ideal when you need to:
- Access a computer behind NAT without port forwarding
- Remotely connect to devices in restricted networks
- Maintain persistent SSH access to devices without public IPs

## Requirements
- A server with a public IP address and root access
- A client machine (the one behind NAT) with internet access
- OpenSSH installed on both machines

## Setup Instructions

### 1. Server Configuration

1.1. Create a dedicated tunnel user:
```bash
adduser \
  --disabled-password \
  --shell /bin/false \
  --gecos "user for reverse ssh tunnel" \
  tunnel-user
```

1.2. Configure SSH server:
- Edit `/etc/ssh/sshd_config`
- Add the following configuration:
```
Match User tunnel-user
   AllowTcpForwarding yes
   X11Forwarding no
   PermitTunnel no
   GatewayPorts yes
   AllowAgentForwarding no
   PermitOpen localhost:2222 your-server.example.com:2222
   ForceCommand echo 'This account is restricted for ssh reverse tunnel use'
```

1.3. Restart SSH service:
```bash
sudo service ssh restart
```

### 2. Client Configuration

2.1. Generate SSH key:
```bash
ssh-keygen -t rsa
```

2.2. Copy the public key:
```bash
cat ~/.ssh/id_rsa.pub
```

### 3. Server Key Installation

3.1. Set up authentication directory:
```bash
cd /home/tunnel-user
mkdir .ssh
touch .ssh/authorized_keys
chmod 700 .ssh
chmod 600 .ssh/authorized_keys
chown -R tunnel-user:tunnel-user .ssh
```

3.2. Add the client's public key to `/home/tunnel-user/.ssh/authorized_keys`

### 4. Testing the Connection

4.1. Test basic connectivity:
```bash
ssh -p 22 -N tunnel-user@your-server.example.com
```

4.2. Test tunnel creation:
```bash
ssh -p 22 tunnel-user@your-server.example.com -N -R your-server.example.com:2222:localhost:22
```

4.3. Test remote access (from another machine):
```bash
ssh -p 2222 client-user@your-server.example.com
```

### 5. Setting Up Auto-reconnection

5.1. Install autossh on the client:
```bash
sudo apt install autossh
```

5.2. Create systemd service:
- Create `/etc/systemd/system/autossh.service`
- Add appropriate configuration (see separate autossh.service file)

5.3. Enable and start the service:
```bash
sudo systemctl enable autossh.service
sudo systemctl start autossh.service
```

5.4 You can check the status:
```bash
systemctl status autossh.service
```

## Troubleshooting

- If password prompt appears during testing, key authentication is not properly set up
- Verify file permissions on `.ssh` directory and files
- Check server logs for connection issues
- Ensure the specified ports are not blocked by firewalls
