Vagrant.configure("2") do |config|
  # Configuración de los dos servidores Apache
  (1..2).each do |i|
    config.vm.define "apache#{i}" do |apache|
      apache.vm.box = "ubuntu/bionic64"
      apache.vm.hostname = "apache#{i}"
      apache.vm.network "private_network", ip: "192.168.50.1#{i}"
      apache.vm.network "forwarded_port", guest: 80, host: "808#{i}"
      apache.vm.provider "virtualbox" do |vb|
        vb.memory = "512"
        vb.cpus = 1
      end
      apache.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y apache2
      SHELL
      apache.vm.synced_folder "./web", "/var/www/html" 
    end
  end

  # Configuración del servidor Nginx (balanceador de carga)
  config.vm.define "nginx" do |nginx|
    nginx.vm.box = "ubuntu/bionic64"
    nginx.vm.hostname = "nginx"
    nginx.vm.network "private_network", ip: "192.168.50.10"
    nginx.vm.network "forwarded_port", guest: 80, host: 8080 
    nginx.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
      vb.cpus = 1
    end
    nginx.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y nginx
      # Configuración de Nginx para balanceo de carga
      cat <<EOF > /etc/nginx/conf.d/load_balancer.conf
      upstream apache_servers {
          server 192.168.50.11;
          server 192.168.50.12;
      }

      server {
          listen 80;

          location / {
              proxy_pass http://apache_servers;
          }
      }
      EOF
      systemctl restart nginx
    SHELL
  end
end
