# VPC's
## What is a VPC
- Virtual Private Cloud
- Virtual network that can be scaled similarly to other AWS features
## Setting up a VPC in AWS
### Creating the VPC
- Click on the "Services" dropdown in the top left of the UI
- Search for VPC and click on it
- On the left hand menu click on "Your VPCs"
- Click the orange "Create VPC" button in the top right to create our VPC
- Give it an appropriate name following your company/team's naming convention
- Give it a IPv4 CIDR block, remember that octets not covered by the CIDR block
must be set to 0 e.g. if you use /16 then the last two octets should be 0
- Make sure the "No IPv6 CIDR block" is selected and that "Tenancy" is Default
- Click "Create VPC" at the bottom of the page
### Creating the IGW
- Gateway that we attach to our VPC to interface with the wider internet
- On the left hand menu click on "Internet Gateways"
- Click the orange "Create Internet gateway" button in the top right to create
our gateway
- Give it an appropriate name following your company/team's naming convention
- Click "Create Gateway" at the bottom of the page
### Connecting the IGW to the VPC
- Now click on the "Actions" dropdown and click "Attach to VPC"
- Find the VPC we just made and click "Connect"
### Creating a Subnet
- A subnet is a smaller section of the overall network, consisting of a range
of IP addresses in your VPC
- On the left hand menu click on "Subnets"
- Click the orange "Create Subnet" button in the top right to create our subnet
- Give it an appropriate name following your company/team's naming convention
- Set the "Availability Zone" to "No preference"
- In the "IPv4 CIDR block" section add the IP of your VPC and edit the host
section (the 0's) to whatever you want. For now we'll just set it up as 1.0
- If you want to add multiple subnets, simply click the "add subnet" button and
follow the same steps before clicking the orange "Create" button in the bottom
right
### Setting up a Route table
- Set of rules used to direct network trafic
- Click on the route table in your VPC details and edit the route table's name
- Click the blue "Create route table" button in the top left
- Give it an appropriate name following your company/team's naming convention
- Find your VPC and select it
- Click the blue "Create" button in the bottom right
- We can then edit the associated subnet and routes for our IP to the IGW
- The finalised route tables for our private and public subnets should look like
 the images below
- Private route:

![Private Route](/images/route_table_private.PNG)
- Public Route:

![Public Route](/images/route_table_public.PNG)

### Setting up NACL
- On the left hand menu click on "Network ACL's"
- Click on the blue "Create network ACL" in the top left
#### Public NACL
- This needs to have the app running, needs internet access in/out (clients),
ssh from our ip (maintenance), to communicate with DB, the ephemeral ports open
- Give it an appropriate name following your company/team's naming convention
- Select your VPC
- Click on the blue "Create" button in the bottom right
- Select your NACL
- Navigate to the Inbound Rules and Outbound rules sections and check that they
are both denied by default
- Now we'll edit the inbound rules for port 80 and 443 (http and https
  respectively) for everyone (0.0.0./0), then port 22 SSH for your IP, and a
  custom TCP rule on port 3000 for your IP
- The rule numbers for these should start at 100 and go up by 10 each time e.g.
100, then 110, then 120 etc (This isn't strictly necessary, just convention and
gives flexibility for adding future rules)
- We'll add one final rule from ports 1024-65535 for everyone so that we can
receive communication back from certain services. This makes the port 3000 rule
unnecessary so we can delete that rule now
- Save these changes with the blue "Save" button in the bottom right
- The finalised setup for the inbound rules should look like the image below

![Pub in](/images/Public_NACL_inbound.PNG)
- Now we'll edit the outbound rules for port 80, ssh on port 22, and ephemeral
ports (1024-65535) from all IP's, as well as sending requests to the private
subnet on the mongod port 20717 for the private subnets IP
- The finalised setup for outbound rules should look like the image below

![Pub out](/images/Public_NACL_outbound.PNG)

#### Private NACL
- This needs to hold our DB, we DON'T want it to have internet access (in) but
we do want it to be able to go to the internet to do updates and get software
etc (out), to be able to receive requests from the public subnet to mongod port,
allow ephemeral ports to respond to db requests, and allow ephemeral in from
public subnet
- Set up inbound rule for port 27017 from the public subnet IP, and ephemeral
ports also from the public subnet IP
- The finalised setup for inbound rules should look like the image below

![Private NACL](/images/Private_NACL_inbound.PNG)
- Set up outbound rules to allow port 80 and 443 for everyone, as well as the
ephemeral ports (1024-65535) for the public subnets IP
- The finalised setup for outbound rules should look like the image below

![Private NACL](/images/Private_NACL_outbound.PNG)

## Creating EC2 instances in our Subnets
- Create an instance like normal
- In "Configuring Instance Details" choose the network we've created and the
desired subnet, enable auto assigning ips and continue
- In "Configuring Security Group" create a new security group as our previous
groups are contained only within their vpc's
- The finalised Security Groups should look like the images below
- App instance:

![App SG](/images/App_SG.PNG)
- DB instance:

![DB SG](/images/DB_SG.PNG)

## Using a Bastion for better Security
- A bastion acts as an additional security measure to make sure our private
networks are as secure as possible
- Allowing someone to ssh into an instance on our private subnet from the public
subnet is not ideal as the public subnet is open to the internet
- Creating a very secure bastion that is very restrictive about what it lets in
and out can be used to access the private subnet and maintain good security
practises
- The Bastion will be its own EC2 instance in our public subnet that we will set
up to only allow an ssh from our IP in its own SG
- We'll then edit the SG for the DB instance to only allow ssh access from
the IP of the Bastion instance
- The security group for the Bastion instance should look like this:

![Bastion SG](/images/Bastion_SG.PNG)


## Getting the nodejs app working
### App
- Copy over the files to the EC2 instance
- SSH into the VM
- Run through the provision file manually but replace the `sudo apt-get install python-software-properties -y`
command with `sudo apt-get install software-properties-common`
- If the next curl command hangs, add an outbound https rule in our public nacl
- Run the curl command again
- Do everything else as usual and the app should be running on port 80!
#### Fixing the posts page
- Run `systemctl status nginx` and see if nginx.service: Failed to parse
- If it did we'll need to run through the following steps
- Create a config file for nginx to read with `printf "[Service]\nExecStartPost=/bin/sleep 0.1\n" > override.conf`
- Make a directory for it go copied into `sudo mkdir /etc/systemd/system/nginx.service.d`
- Copy over the file to this new directory `sudo cp override.conf /etc/systemd/system/nginx.service.d/override.conf`
- Reload the daemon `sudo systemctl daemon-reload`
- Stop and start nginx with `sudo systemctl stop nginx` and `sudo systemctl start nginx`
- Start the app with pm2 and posts should work (it still won't work with npm
for some reason but pm2 should suffice)
### DB without Bastion
- Set up new EC2 instance as before but with the subnetwork as private
- Edit the private route table to allow a route from your IP to the Internet
Gateway that we set up earlier, we'll also have to open it up to 0.0.0.0/0 for
the provisioning script to work since apt uses https and we need to accept this
traffic
- SSH into the instance and run through provision file for the db
### DB with Bastion
- Follow the steps from the above section but ssh into the instance from within
the Bastion instance
