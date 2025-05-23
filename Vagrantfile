# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

  config.vm.provider "virtualbox" do |vb|
    vb.name   = "gitlab-https"
    vb.memory = 8192
    vb.cpus   = 4
  end

  config.vm.network "private_network", type: "dhcp"
  config.vm.network "forwarded_port", guest: 443, host: 8443, auto_correct: true
  config.vm.synced_folder ".", "/vagrant"

  config.vm.provision "shell", privileged: true, inline: <<-SHELL
    set -eux

    echo "==> Mise à jour du système"
    apt-get update -y && apt-get upgrade -y

    echo "==> Installation des dépendances"
    apt-get install -y curl ca-certificates tzdata openssh-server gnupg lsb-release

    echo "==> Installation du dépôt GitLab CE"
    curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | bash

    echo "==> Génération du certificat auto-signé AVEC SAN"
    IP=$(hostname -I | grep -oE '192\\.168\\.56\\.[0-9]+')
    mkdir -p /etc/gitlab/ssl
    chmod 700 /etc/gitlab/ssl

    cat > /etc/gitlab/ssl/${IP}.cnf <<EOF
[req]
default_bits       = 2048
prompt             = no
default_md         = sha256
req_extensions     = req_ext
distinguished_name = dn

[dn]
C = FR
ST = Dev
L = Local
O = DevOps
CN = ${IP}

[req_ext]
subjectAltName = @alt_names

[alt_names]
IP.1 = ${IP}
EOF

    openssl req -new -x509 -nodes -days 365 \
      -newkey rsa:2048 \
      -keyout /etc/gitlab/ssl/${IP}.key \
      -out /etc/gitlab/ssl/${IP}.crt \
      -config /etc/gitlab/ssl/${IP}.cnf

    echo "==> Configuration GitLab pour HTTPS avec SAN"
    cat > /etc/gitlab/gitlab.rb <<EOF
external_url "https://${IP}"
nginx['ssl_certificate'] = "/etc/gitlab/ssl/${IP}.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/${IP}.key"
letsencrypt['enable'] = false
EOF

    echo "==> Installation et reconfiguration de GitLab CE"
    EXTERNAL_URL="https://${IP}" apt-get install -y gitlab-ce
    gitlab-ctl reconfigure

    echo "==> Affichage du mot de passe root généré"
    cat /etc/gitlab/initial_root_password || true

    echo "==> Ajout de la clé SSH à l'utilisateur root"
    if [ -f /vagrant/gitlab_id_ed25519.pub ]; then
      PUB_KEY=$(tr -d '\\n' < /vagrant/gitlab_id_ed25519.pub)
      gitlab-rails runner "u = User.find_by(username: 'root'); if u; u.keys.create!(title: 'root-key', key: '$PUB_KEY'); else; puts 'Utilisateur root introuvable'; end"
    else
      echo "⚠️  Clé publique non trouvée : /vagrant/gitlab_id_ed25519.pub"
    fi

    echo "==> GitLab est prêt : https://${IP}"
    echo "    Utilisateur : root (mot de passe affiché ci-dessus)"
  SHELL
end
