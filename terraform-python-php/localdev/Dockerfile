FROM registry.access.redhat.com/ubi10/ubi:10.2-1790572136

# Basic customization
# ==========================================================================================
RUN <<-'EOT'
    dnf -y install zsh util-linux-user zip unzip git make yum-utils
    dnf clean all
    chsh -s /bin/zsh
    curl --create-dirs --output-dir ~/.zsh/ -O \
    "https://raw.githubusercontent.com/git/git/master/contrib/completion/{git-prompt.sh,git-completion.bash}"
    curl --create-dirs --output-dir ~/.zsh/ -o _git \
    "https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.zsh"
EOT

COPY <<-'EOF' /root/.zshrc
    # Initialize ZSH completions
    autoload -Uz compinit && compinit

    # Source Git completion and Git prompt scripts
    zstyle ':completion:*:*:git:*' script ~/.zsh/git-completion.bash
    . ~/.zsh/git-prompt.sh

    # Set Git prompt
    GIT_PS1_SHOWDIRTYSTATE=1
    GIT_PS1_SHOWSTASHSTATE=1
    GIT_PS1_SHOWUNTRACKEDFILES=1
    GIT_PS1_SHOWUPSTREAM="auto"

    # Set ZSH Prompt
    setopt PROMPT_SUBST;
    PS1='%F{green}%n@%m%f:%F{yellow}%c%F{red}%B$(__git_ps1 " (%s)")%b%f\$ '

    # Enable CLI color
    export CLICOLOR=1

    # Aliases
    alias ll="ls -la --color=auto"
EOF

# Repositories
# ==========================================================================================
RUN <<-'EOT'
    dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-10.noarch.rpm
    dnf -y install https://dev.mysql.com/get/mysql84-community-release-el10-3.noarch.rpm
    dnf config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
    dnf clean all
EOT

# Packages
# ==========================================================================================
RUN <<-'EOT'
    curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    ./aws/install
    dnf -y install jq nodejs httpd mysql-community-server phpmyadmin terraform-1.16.4-1 packer-1.16.1-1
    dnf clean all
EOT

# Python
# ==========================================================================================
RUN <<-'EOT'
    dnf -y install python3 python3-pip
    dnf clean all
    pip --no-cache-dir install ansible
EOT

# PHP
# ==========================================================================================
SHELL ["/bin/zsh", "-o", "pipefail", "-c"]
RUN <<-'EOT'
    dnf -y install php
    dnf clean all
    # PHP FPM configuration
    sed -i 's/;listen.owner = nobody/listen.owner = apache/g' /etc/php-fpm.d/www.conf
    sed -i 's/;listen.group = nobody/listen.group = apache/g' /etc/php-fpm.d/www.conf
    sed -i 's/listen.acl_users = apache/;listen.acl_users = apache/g' /etc/php-fpm.d/www.conf
    # PHP Composer
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
    # PHP Codesniffer
    composer global require "squizlabs/php_codesniffer=*"
EOT

ENV PATH="$PATH:/root/.config/composer/vendor/bin"
# PHP Compatibility
WORKDIR /root
RUN <<-'EOT'
    curl https://github.com/PHPCompatibility/PHPCompatibility/archive/refs/tags/9.3.5.zip -O temp.zip
    unzip temp.zip
    rm temp.zip
    phpcs --config-set installed_paths /root/PHPCompatibility-9.3.5
EOT

# Configuration
# ==========================================================================================
RUN <<-'EOT'
    systemctl enable httpd mysqld php-fpm
    # Make HTML document root point to /code
    rm -rf /var/www/html
    ln -s /code /var/www/html
    # PHPMyAdmin : Allow access from all IPs
    # sed -i '/^<Directory \/usr\/share\/phpMyAdmin\/>/a Require all granted' /etc/httpd/conf.d/phpMyAdmin.conf
EOT

# Secrets
# ==========================================================================================
# Wildcard suffix will ignore errors if file or dir doesn't exist.
COPY secrets/.ssh* /root/.ssh
COPY secrets/.aws* /root/.aws
COPY secrets/.gitconfig* /root/.gitconfig
RUN chmod -R 0600 /root/.ssh

# Final
# ==========================================================================================
VOLUME /code
WORKDIR /code
EXPOSE 80

ENTRYPOINT ["/usr/sbin/init"]
CMD ["zsh"]
