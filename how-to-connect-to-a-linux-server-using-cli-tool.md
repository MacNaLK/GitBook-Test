---
description: page description here
cover: >-
  https://images.unsplash.com/photo-1629654291663-b91ad427698f?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwzfHxsaW51eHxlbnwwfHx8fDE3MzY2ODY0NDR8MA&ixlib=rb-4.0.3&q=85
coverY: 50
layout:
  cover:
    visible: true
    size: full
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# How to Connect to a Linux Server using CLI tool

This was my fisfirst note

Remember to ensure that SSH is enabled and that you have the necessary permissions to access the server. Additionally, for enhanced security, consider using SSH key-based authentication instead of passwords. To set up SSH keys:

To connect to a Linux server using SSH, use the following command in your terminal:

```sh
ssh username@server_ip_address
```

Replace `username` with your actual username and `server_ip_address` with the server's IP address. If you have a specific port for SSH, include it with the `-p` option:

```sh
ssh username@server_ip_address -p port_number
```

For enhanced security, use SSH key-based authentication instead of passwords.

```
sudo su
```

