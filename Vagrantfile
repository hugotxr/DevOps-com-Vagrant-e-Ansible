# -*- mode: ruby -*-
# vi: set ft=ruby :


nome_equipe = "hugo.anolivia" 
ip_arq_suffix = "15"
ip_db_suffix = "04"
ip_app_suffix = "54"
host_only_ip = "192.168.56.1" 

Vagrant.configure("2") do |config|
  
    config.vm.box = "debian/bookworm64"
    config.ssh.insert_key = false 

    config.vm.provider "virtualbox" do |vb|
    vb.linked_clone = true 
    vb.check_guest_additions = false 
    vb.memory = "512" # Memória padrão
  end
  
    config.vm.network "private_network", 
    ip: host_only_ip, 
    virtualbox__intnet: true,     virtualbox__hostonly: host_only_ip,
    virtualbox__netmask: "255.255.255.0",
    virtualbox__dhcp: false # Desabilita o DHCP nativo do VirtualBox para essa rede

   
  # --- 1. Servidor de Arquivos (arq) - IP Fixo ---
    config.vm.define "arq" do |arq|
    arq.vm.hostname = "arq.#{"hugo.anolivia"}.devops"
    
    # Define o IP Fixo: 192.168.56.115 na Host-Only Network criada acima
    arq.vm.network "private_network", type: "dhcp"     
    arq.vm.network "private_network", 
      auto_config: false,       
      virtualbox__intnet: true,       
      mac: "0800270000AA" 
    # Reaplicação da memória correta para ARQ
    arq.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
      
      # DISCOS ADICIONAIS: 3 discos de 10 GB para LVM
      (1..3).each do |i|
        disk_name = "arq-disk#{i}.vdi"
        vb.customize ['createhd', '--filename', disk_name, '--size', 10 * 1024]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', i + 1, '--device', 0, '--type', 'hdd', '--medium', disk_name]
      end
    end
  end

    # --- 2. Servidor de Banco de Dados (db) - IP via DHCP (Estático por MAC) ---
    config.vm.define "db" do |db|
      db.vm.hostname = "db.#{"hugo.anolivia"}.devops"
      # Rede interna sem autoconfig, esperando DHCP do ARQ.
      db.vm.network "private_network", auto_config: false, virtualbox__intnet: true, mac: "0800270000DB"
    end
  
    # --- 3. Servidor de Aplicação (app) - IP via DHCP (Estático por MAC) ---
    config.vm.define "app" do |app|
      app.vm.hostname = "app.#{nome_equipe}.devops"
      # Rede interna sem autoconfig, esperando DHCP do ARQ.
      app.vm.network "private_network", auto_config: false, virtualbox__intnet: true, mac: "0800270000AP"
    end
  
    # --- 4. Host Cliente (cli) - IP via DHCP (Dinâmico) ---
    config.vm.define "cli" do |cli|
      cli.vm.hostname = "cli.#{"hugo.anolivia"}.devops"
      cli.vm.provider "virtualbox" do |vb|
        vb.memory = "1024" # RAM maior
      end
    # Obtém IP via DHCP (dinâmico), esperando DHCP do ARQ.
    cli.vm.network "private_network", auto_config: false, virtualbox__intnet: true
  end
  
    # === PROVISIONAMENTO COM ANSIBLE ===
    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "site.yml"
      ansible.inventory_path = "hosts.ini"
      ansible.limit = "all"
      ansible.verbose = true
      ansible.compatibility_mode = "auto"
    end
end
