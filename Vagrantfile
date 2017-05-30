# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"
BOX_URL = "../Facilities/vpshere/dummy.box"
VAGRANT_KEY = "../Facilities/vagrant.key"


#pour éviter de versionner avec le mdp, un fichier externe défini cette constante
#Ce fichier externe est ignoré du système SCM
#Sinon décomenter la ligne et utiser une variable d'environement ou mettre le mdp dans le fichier
#Ne PAS commiter le fichier avec un mdp.
#VSPHERE_PASSWORD=ENV['vsphere_password']
external = File.read '../Vsphere_password'
eval external # load vsphere password

VSPHERE_USER='account@domain'
VSPHERE_HOST = 'vcenter_hostname'
VSPHERE_DATA_CENTER_NAME = 'Data center name'
VSPHERE_CLUSTER_NAME = 'cluster name'
VSPHERE_TEMPLATE_NAME = 'Templates/Ubuntu14-Template-App-Updated'
VSPHERE_CLONE_FROM_VM = true
VSPHERE_CUSTOMIZATION_SPEC_NAME = 'Linux Ip Fixe'
VSPHERE_INSECURE = true

CHEF_CHEF_SERVER_URL = "https://chef-server.domain.local"
CHEF_VALIDATION_KEY_PATH = "c:/d/chef/confs/chef-server/chef-server.pem"
#CHEF_ENVIRONMENT = "I1"
CHEF_DELETE_NODE = true
CHEF_DELETE_CLIENT = true
#CHEF_HTTP_PROXY = 'http://proxy:9090'
#CHEF_NO_PROXY = 'p1frchef01,nexus,localhost,172.30.,172.30.,.nantes.cabinet-besse.fr'

envToPath = {
  'D1' => 'D1_Developpement',
  'I1' => 'I1_Integration',
  'R1' => 'R1_Recette_cycle_court',
  'R2' => 'R2_Recette_cycle_long',
  'R3' => 'R3_Pre_production',
  'P1' => 'P1_Production',
  'T1' => 'T1_Test',
  'F1' => 'F1_Formation',
  'Z1' => 'Z1_Multi_environnements'
}

envToCust = {
  'D1' => 'non prod',
  'I1' => 'non prod',
  'R1' => 'non prod',
  'R2' => 'non prod',
  'R3' => 'non prod',
  'P1' => 'prod',
  'T1' => 'non prod',
  'F1' => 'non prod',
  'Z1' => 'non prod'
}

envToVlan = {
  'D1' => 'PRF-VLAN004',
  'I1' => 'PRF-VLAN004',
  'R1' => 'PRF-VLAN004',
  'R2' => 'PRF-VLAN004',
  'R3' => 'PRF-VLAN004',
  'P1' => 'PRF-VLAN002',
  'T1' => 'PRF-VLAN004',
  'F1' => 'PRF-VLAN004',
  'Z1' => 'PRF-VLAN004'
}

machinesBases = {
  'machine1' => {
    'ip_address' => '172.xxx.xxx.5',
    'env' => 'R1',
    'roles' => ['elasticsearch_server'],
    'additionnal_recipes' => ['cbp-elasticsearchv5']
  },
  'r1fr2els03' => {
    'ip_address' => '172.xxx.xxx.6',
    'env' => 'R1',
    'roles' => ['elasticsearch_server'],
    'additionnal_recipes' => ['cbp-elasticsearchv5']
  },
  'r1frkib03' => {
    'ip_address' => '172.xxx.xxx.7',
    'env' => 'R1',
    'roles' => ['elasticsearch_server'],
    'additionnal_recipes' => ['cbp-elasticsearchv5','cbp-kibanav5', 'cbp-logstashv5']
  }
}


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "dummy"
  config.vm.box_url = BOX_URL
  config.ssh.private_key_path = VAGRANT_KEY
  config.vm.synced_folder ".", "/vagrant", disabled: true

  machinesBases.each do |hostname,properties|
    config.vm.define hostname do |box|
      box.vm.provider :vsphere do |vsphere|
        vsphere.host = VSPHERE_HOST
        vsphere.data_center_name = VSPHERE_DATA_CENTER_NAME
	vsphere.compute_resource_name = ""
        vsphere.template_name = VSPHERE_TEMPLATE_NAME
        vsphere.clone_from_vm = VSPHERE_CLONE_FROM_VM
        vsphere.customization_spec_name = "#{VSPHERE_CUSTOMIZATION_SPEC_NAME} #{envToCust[properties['env']]}"
        vsphere.insecure = VSPHERE_INSECURE
        vsphere.user = VSPHERE_USER
        vsphere.password = VSPHERE_PASSWORD
        vsphere.vm_base_path = envToPath[properties['env']]
        vsphere.vlan = envToVlan[properties['env']]

        vsphere.name = hostname
      end
      box.vm.hostname = "#{hostname}.domain.local"

      box.vm.network :private_network, ip: properties["ip_address"]
      box.vm.provision :chef_client do |chef|
        chef.chef_server_url = CHEF_CHEF_SERVER_URL
        chef.validation_key_path = CHEF_VALIDATION_KEY_PATH
        chef.environment = properties["env"]
        chef.delete_node = CHEF_DELETE_NODE
        chef.delete_client = CHEF_DELETE_CLIENT
        chef.binary_env = "SSL_CERT_FILE=/opt/chef/embedded/ssl/certs/cacert.pem"
        #chef.http_proxy = CHEF_HTTP_PROXY
        #chef.https_proxy = CHEF_HTTP_PROXY
        #chef.no_proxy = CHEF_NO_PROXY
        chef.encrypted_data_bag_secret_key_path  = "c:/d/chef/confs/chef-server/data_bag_secret.key"

        chef.add_recipe "base-linux"
        properties['roles'].each do |role|
          chef.add_role role
        end
        properties['additionnal_recipes'].each do |recipe|
          chef.add_recipe recipe
        end
      end
    end
  end unless machinesBases.length == 0
end
