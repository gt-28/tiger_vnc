# How to configure Virtual Network Computing (VNC) in Red Hat Enterprise Linux 7 / CentOS

## Configuring `tigervnc-server` on `rhel/centos`

## Pre-requisites:

Update CentOS System and Create a Linux User.
Install `gnome` or `kde` desktop environments
Install XFCE Desktop and `TigerVNC`.
Initiallize VNC Configuration.
Configure TigerVNC.
Run TigerVNC as a Service.
Connect to the VNC Server Through SSH Tunnel/VNC-viewer.

### Steps:

1. Update system:

    ``` 
    yum update -y
    ```
2. install required GUI packages

    ```
    yum groupinstall gnome-desktop x11 fonts -y
    ```

3. Install `TigerVNC`

    ```
    yum -y install tigervnc-server tigervnc
    ```

4. Modify the `tigervnc` config file to reflect user settings.

> Step 1:

   `cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service`
   
   
> Notes: https://bugzilla.redhat.com/show_bug.cgi?id=1583159

    For that matter, ^^ here's our unit file for a VNC server that runs as a "service":

 * Start and Stop are properly detected
 * Logging out of the server causes the service to be marked as stop
 * 15 seconds after logout, the service is restart to permit a new login

> The fix is to remove the `runuser` invocation from the ExecStart= key, start vncserver directly and instead set the User= and Group= keys in the service section:

Edit the file to look like this:

    ```
    [Unit]
    Description=Remote desktop service (VNC)
    After=syslog.target network.target

    [Service]

    # Newer desktop environments know sd_notify, so we can use that
    Type=notify
    NotifyAccess=all

    WorkingDirectory=/home/ec2-user
    User=ec2-user
    Group=ec2-user

    # Clean any existing files in /tmp/.X11-unix environment
    ExecStartPre=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'
    #ExecStart=/usr/sbin/runuser -l ec2-user -c "/usr/bin/vncserver %i"
    ExecStart=/usr/bin/vncserver %i
    PIDFile=/home/ec2-user/.vnc/%H%i.pid
    ExecStop=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'

    # After exit, we want our service to restart
    Restart=on-success
    RestartSec=15


    [Install]
    WantedBy=multi-user.target

    ```


5. start the process:

    ```
   systemctl daemon-reload
   systemctl enable vncserver@:1.service
   systemctl start vncserver@:1.service
   systemctl status vncserver@:1.service
   ```
