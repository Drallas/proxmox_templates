
# README.md - Proxmox VM Deployment met Ansible

Dit project (colab met Grok) helpt je om VM's (***virtuele machines***) in Proxmox te maken en te beheren met Ansible. Het bestaat uit twee playbooks: één om een template te maken (```create_template.yml```) en één om VM's te deployen vanuit dat template (```deploy_template.yml```). We gebruiken DHCP voor IP-toewijzing, maar vertrouwen op vaste IP’s uit de inventory die je DHCP-server moet reserveren. Geen jmespath nodig, alles blijft simpel en werkt idempotent.

## Hoe gebruik je dit?

### Wat je nodig hebt

- **Proxmox-server**: Zorg dat je Proxmox draait en SSH-toegang hebt.

- **Ansible**: Installeer Ansible op je MacBook (brew install ansible of pip install ansible).

- **SSH-sleutel**: Een SSH-sleutel (bijv. ~/.ssh/id_ed25519) om in te loggen op Proxmox en de VM's.

- **Inventory**: Het bestand inventory.yml met je Proxmox-host en VM-details (zie hieronder).

### Stap 1: Template maken

Dit playbook maakt een basis-VM-template in Proxmox die je later kunt clonen.

1.Run het commando:

```bash
ansible-playbook -i inventory.yml create_template.yml
```

2.Wat het doet:

- Maakt een VM met ID 9000 (pas dit aan in create_template.yml als je een andere ID wilt).
- Installeert een minimale Debian-installatie en wat handige tools.
- Zet het om naar een template.

### Stap 2: VM's deployen

Dit playbook kloont het template en zet je VM's live.

1.Run het commando:

```bash
ansible-playbook -i inventory.yml deploy_template.yml
```

2.Wat het doet:

- Checkt of de VM's al bestaan (bijv. ID 105, 106, 107) met qm list.
- Als ze niet bestaan: kloont het template, stelt MAC-adressen in, gebruikt DHCP, en start ze.
- Pingt de VM's op de IP's uit je inventory om te checken of ze bereikbaar zijn.
- Installeert de qemu-guest-agent en zorgt dat die draait.
- Geeft een net overzicht van je VM's in een Markdown-achtige stijl.

3.Voorbeeld output:

```bash
TASK [deploy_template : Display successful installations]
ok: [proxmox] => {
    "successful_vms": [
        {
            "id": 105,
            "ip": "192.168.1.111",
            "mac": "BC:24:11:85:68:0C",
            "name": "dswarm01"
        },
        {
            "id": 106,
            "ip": "192.168.1.112",
            "mac": "BC:24:11:38:39:D6",
            "name": "dswarm02"
        },
        {
            "id": 107,
            "ip": "192.168.1.113",
            "mac": "BC:24:11:BB:19:A5",
            "name": "dswarm03"
        }
    ]
}
```

Dit kun je lezen als:

```bash
## Succesvolle installaties
- VM dswarm01 (ID: 105)
  - IP: 192.168.1.111  
  - MAC: BC:24:11:85:68:0C  
  - Login: `ssh debian@192.168.1.111`  
- VM dswarm02 (ID: 106)
  - IP: 192.168.1.112  
  - MAC: BC:24:11:38:39:D6  
  - Login: `ssh debian@192.168.1.112`  
- VM dswarm03 (ID: 107)
  - IP: 192.168.1.113  
  - MAC: BC:24:11:BB:19:A5  
  - Login: `ssh debian@192.168.1.113`  
```

## Inventory aanpassen

Het bestand inventory.yml vertelt Ansible waar je Proxmox-server staat en welke VM's je wilt maken. Hier een voorbeeld:

```yaml
all:
  hosts:
    proxmox:
      ansible_host: 192.168.1.10  # Jouw Proxmox IP
      ansible_user: root          # Gebruiker voor Proxmox
      ansible_ssh_private_key_file: ~/.ssh/id_ed25519
  vars:
    vmid: 9000                  # Template ID
    base_clone_id: 105          # Eerste VM ID voor cloning
    storage: local              # Waar de VM's opgeslagen worden
    servers:
      - name: dswarm01
        mac: BC:24:11:85:68:0C
        ip: 192.168.1.111       # Verwachte IP (moet matchen met DHCP-reservering)
      - name: dswarm02
        mac: BC:24:11:38:39:D6
        ip: 192.168.1.112
      - name: dswarm03
        mac: BC:24:11:BB:19:A5
        ip: 192.168.1.113
```

- Pas ```ansible_host``` aan naar je Proxmox IP.
- Zorg dat je DHCP-server deze IP's reserveert op basis van de MAC-adressen, anders faalt de ping-check.

## Problemen oplossen

- Ping faalt: Controleer of je DHCP-server de juiste IP's geeft:

```bash
ping 192.168.1.111
```

- SSH werkt niet: Test handmatig:

```bash
ssh -i ~/.ssh/id_ed25519 debian@192.168.1.111
```

- VM's worden niet gekloond: Check de ID's:

```bash
qm list
````
