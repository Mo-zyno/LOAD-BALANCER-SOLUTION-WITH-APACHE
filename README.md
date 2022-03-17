# Load-Balancer-Solution-With-Apache

## THIS IS A 3TIER WEB APPLICATION DESIGN WITH A SINGLE DATABASE, AN NFS SERVER AS SHARED FILE STORAGE PLUS A LOAD BALANCER.

When we access a website in the Internet we use an URL and we do not really know how many servers are out there serving our requests, this complexity is hidden from a regular user, but in case of websites that are being visited by millions of users per day (like Google or Reddit) it is impossible to serve all the users from a single Web Server.

In this project we will enhance our Tooling Website solution by adding a Load Balancer to disctribute traffic between Web Servers and allow users to access our website using a single URL.

Task
Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 intance. Make sure that users can be served by Web servers through the Load Balancer.

To simplify, let us implement this solution with 2 Web Servers, the approach will be the same for 3 and more Web Servers.

*Prerequisites*

1.Two RHEL8 Web Servers

2.One MySQL DB Server (based on Ubuntu 20.04)

3.One RHEL8 NFS server
