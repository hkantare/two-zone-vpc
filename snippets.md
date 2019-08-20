## Dnsimple config
```
variable dnsimple_token {}
variable dnsimple_account {}
provider "dnsimple" {
  token   = "${var.dnsimple_token}"
  account = "${var.dnsimple_account}"
}

resource "dnsimple_record" "elk" {
  domain = "${var.domain}"
  name   = "elk"
  value  = "${ibm_compute_vm_instance.elk_node.ipv4_address}"
  type   = "A"
  ttl    = 3600
}

resource "dnsimple_record" "minio_domain" {
  count  = "${var.node_count}"
  domain = "${var.domainname}"
  name   = "minionode${count.index+1}"
  value  = "${element(ibm_compute_vm_instance.minionodes.*.ipv4_address_private,count.index)}"
  type   = "A"
  ttl    = 300
}
```

## Ansible inventory

```
resource "ansible_group" "web" {
  depends_on = ["ibm_compute_vm_instance.node"]
  inventory_group_name = "web"
}

resource "ansible_host" "node1_hostentry" {
  depends_on = ["ansible_group.web"]
    inventory_hostname = "node-1"
    groups = ["web"]
    vars {
        ansible_host = "${ibm_compute_vm_instance.node.0.ipv4_address}"
        ansible_user = "ryan"
    }
}

resource "ansible_host" "node2_hostentry" {
  depends_on = ["ansible_host.node1_hostentry"]
    inventory_hostname = "node-2"
    groups = ["web"]
    vars {
        ansible_host = "${ibm_compute_vm_instance.node.1.ipv4_address}"
        ansible_user = "ryan"
    }
}
```

## Local file 
```
resource "local_file" "output" {
  content = <<EOF
${jsonencode(ibm_service_key.es_service_key.credentials)}"
EOF

  filename = "${path.module}/${terraform.workspace}.env"
}
```

## data /org /space 

```hcl
data "ibm_org" "orgData" {
  org = "${var.ibm_bmx_org}"
}

data "ibm_account" "accountData" {
  org_guid = "${data.ibm_org.orgData.id}"
}

data "ibm_space" "spaceData" {
  space = "${var.ibm_bmx_space}"
  org   = "${var.ibm_bmx_org}"
}

```

## NO OS Server

```
resource "ibm_compute_bare_metal" "monthly_bm1" {
    package_key_name = "DUAL_E52600_4_DRIVES"
    process_key_name = "INTEL_XEON_2620_2_40"
    memory = 64
    os_key_name = "OS_NO_OPERATING_SYSTEM"
    hostname = "no-os"
    domain = "${var.domainname}"
    datacenter = "${var.datacenter["us-south3"]}"
    network_speed = 10000
    public_bandwidth = 500
    disk_key_names = [ "HARD_DRIVE_4_00_TB_SATA" ]
    hourly_billing = false
    private_network_only = false
    unbonded_network = true
    tags = [
      "ryantiffany",
    ]
    redundant_power_supply = true
    public_vlan_id = "${var.pub_vlan["us-south3"]}"
    private_vlan_id = "${var.priv_vlan["us-south3"]}"
    ssh_key_ids = ["${data.ibm_compute_ssh_key.deploysshkey.id}"]
}
```