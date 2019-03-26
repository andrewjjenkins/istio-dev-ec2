Building Istio on your laptop causing your fans to scream?  Pains building on
Mac or Windows?  This CloudFormation template will quickly spin up an EC2
virtual machine capable of building Istio and Envoy.

To create the stack:

    git clone https://github.com/andrewjjenkins/istio-dev-ec2
    cd istio-dev-ec2
    STACKNAME=andrew-istio-dev KEYNAME=andrews-ssh-key ./create-istio-build.sh


`STACKNAME` is what you would like the cloudformation stack to be named.  This
stack will have all the resources (like the EC2 instance) inside of it, so you
should delete it when you're done.  Defaults to `istio-dev`

`KEYNAME` is the name of an SSH keypair in AWS that you need to configure and
supply.  See [AWS
docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
for more.  No default.  You can get a list of configured keypairs using `aws
ec2 describe-key-pairs`.

When done, output like this:

    {
      "StackId": "arn:aws:cloudformation:us-west-2:112345677999:stack/andrew-istio-dev/64444444-2baa-1111-bbbb-dddddddddddd"
    }
    Stack created.
    Login with ssh command:
      ssh ubuntu@34.201.122.297
    Delete the stack when you're done with:
      aws cloudformation delete-stack --stack-name andrew-istio-dev

# In the box

 - Default instance is an m4.2xlarge with 60GB EBS
 - Minikube and docker are installed and running
 - Kubernetes utils like kubectl, kubectx, kubens, stern, helm installed
 - Build toolchain like build-essential, golang, git, jq, tmux, clang, bazel installed

# What now?

If you would like to use your local SSH credentials to clone from github, you
should do SSH agent forwarding instead of copying your public key.  A
development workflow might look like:

    ssh -A ubuntu@34.201.122.297
    cd go/src/istio.io
    git clone git@github.com:istio/istio.git
    cd istio
    make
    make docker.all test/local/auth/e2e_simple

# Local editors

If you want to use your local editor, a great option is sshfs.  You install the
sshfs client locally on your machine, and then you can mount the clone on the
remote machine and edit files.  If your local machine is a mac:

    brew cask install osxfuse
    brew install sshfs
    sshfs ubuntu@34.201.122.297:go/src istio-go-src

When done, unmount it before tearing down the stack:

    umount istio-go-src

# Stopping to save money

If you shutdown the instance, it will enter the EC2 stopped state but not
terminate.  While this is happening, you don't have to pay the EC2 instance
charges but you will still have to pay for the storage and Elastic IP.

    ssh ubuntu@34.201.122.297 "shutdown -h now"

Start back up:

    aws ec2 start-instances --instance-ids `aws cloudformation describe-stacks --stack-name andrew-istio-dev | jq -r '.Stacks[0].Outputs[] | select(.OutputKey=="InstanceId") | .OutputValue'`

# Done

Remember to delete the stack when you're done so you don't accumulate charges:

    aws cloudformation delete-stack --stack-name istio-dev
