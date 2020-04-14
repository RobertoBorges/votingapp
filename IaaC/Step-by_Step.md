aksLocation=<Azure region>
resGroup=<Resource groupe name>
vnet=<VNet Name>
vnetSub=<SbNet for AKS name>
VNSubnet=<SbNet for Virtual Nodes name>
SPk8s=<Service Principal name>
KUBE_DNS_IP="10.0.0.10"
DOCKER_BRIDGE_ADDRESS="172.17.0.1/16"
AKSCluster=<ASK Cluster name>

# This is holding all the resources for our cluster

az group create \
    -l $aksLocation \
    -n $resGroup

# Also we create a subnet for our cluster

az network vnet create \
    --resource-group $resGroup \
    --name $vnet \
    --address-prefixes 10.0.0.0/8  \
    --subnet-name $vnetSub \
    --subnet-prefix 10.240.0.0/16

# Create a subnet for virtual node

az network vnet subnet create \
    --resource-group $resGroup  \
    --vnet-name $vnet \
    --name $VNSubnet \
    --address-prefix 10.241.0.0/16 

# The service principal allows us to create other cloud resources

az ad sp create-for-rbac \ 
    --name $SPk8s \ 
    –-role  Contributor 

# Create the CLuster AKS with all ADDONs
az aks create \
    --resource-group $resGroup \
    --name $AKSCluster \
    --node-count 3 \
    --service-principal <SPN ID> \
    --client-secret <SPN Secret> \
    --generate-ssh-keys \
    --network-plugin azure \
    --dns-service-ip $KUBE_DNS_IP \
    --docker-bridge-address $DOCKER_BRIDGE_ADDRESS \
    --vnet-subnet-id <VNET AKS ID> \
    --load-balancer-sku standard \
    --enable-vmss \
    --node-zones 1 2 3 \
    --network-policy calico

az aks enable-addons\
    -g $resGroup\
    -n $AKSCluster\
    --addons virtual-node\
    --subnet-name $VNSubnet