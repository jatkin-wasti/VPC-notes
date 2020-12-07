# VPC's
## What is a VPC

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
- On the left hand menu click on "Internet Gateways"
- Click the orange "Create Internet gateway" button in the top right to create
our gateway
- Give it an appropriate name following your company/team's naming convention
- Click "Create Gateway" at the bottom of the page
### Connecting the IGW to the VPC
- Now click on the "Actions" dropdown and click "Attach to VPC"
- Find the VPC we just made and click "Connect"
### Creating a Subnet
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
- Click on the route table in your VPC details and edit the route table's name
- Click the blue "Create route table" button in the top left
- Give it an appropriate name following your company/team's naming convention
- Find your VPC and select it
- Click the blue "Create" button in the bottom right

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
- Now we'll edit the outbound rules for port 80, ssh on port 22, and ephemeral
ports (1024-65535) from all IP's, as well as sending requests to the private
subnet on the mongod port 20717 for the private subnets IP
#### Private NACL
- This needs to hold our DB, we DON'T want it to have internet access (in) but
we do want it to be able to go to the internet to do updates and get software
etc (out), to be able to receive requests from the public subnet to mongod port,
allow ephemeral ports to respond to db requests, and allow ephemeral in from
public subnet
