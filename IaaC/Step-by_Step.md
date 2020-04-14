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
--service-principal e68976d2-f413-4625-af16-686ed2a6ad2e \
--client-secret 2b33ebce-c7f1-4551-aec3-d249cd9506b4 \
--generate-ssh-keys \
--network-plugin azure \
--dns-service-ip $KUBE_DNS_IP \
--docker-bridge-address $DOCKER_BRIDGE_ADDRESS \
--vnet-subnet-id /subscriptions/475db780-ab6b-4ebb-8fda-1e9b74bc2c23/resourceGroups/aksdemo-rg/providers/Microsoft.Network/virtualNetworks/aksvnet/subnets/virtualnosubnet \
--load-balancer-sku standard \
--enable-vmss \
--node-zones 1 2 3 \
--network-policy calico

az aks enable-addons \
--resource-group $resGroup \
--name $AKSCluster \
--addons virtual-node \
--subnet-name $VNSubnet
