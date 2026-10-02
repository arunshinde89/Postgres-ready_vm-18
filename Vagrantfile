Vagrant.configure("2") do |config|

  # ============================================================
  # BASE OS
  # ============================================================

  config.vm.box = "rockylinux/8"

  # Hostname
  config.vm.hostname = "postgres-vm"


  # ============================================================
  # NETWORK
  # ============================================================

  # Fixed private IP
  # Use this for PuTTY, PostgreSQL, pgAdmin etc.
  config.vm.network "private_network",
    ip: "192.168.136.10"

  # Vagrant/VMware will retain its normal NAT interface
  # for internet access.


  # ============================================================
  # VMWARE
  # ============================================================

  config.vm.provider "vmware_desktop" do |v|

    # Open VMware console
    v.gui = true

    # RAM = 4 GB
    v.vmx["memsize"] = "4096"

    # CPU = 2
    v.vmx["numvcpus"] = "2"

  end


  # ============================================================
  # PROVISIONING
  # ============================================================

  config.vm.provision "shell", inline: <<-'SHELL'

    echo "======================================"
    echo " Starting PostgreSQL Lab Provisioning "
    echo "======================================"


    # ==========================================================
    # 1. UPDATE OS
    # ==========================================================

    echo "Updating operating system..."

    dnf update -y


    # ==========================================================
    # 2. INSTALL REQUIRED OS PACKAGES
    # ==========================================================

    echo "Installing required packages..."

    dnf install -y xfsprogs


    # ==========================================================
    # 3. ROOT PASSWORD
    # ==========================================================

    echo "Setting root password..."

    echo 'root:welcome1' | chpasswd


    # ==========================================================
    # 4. ENABLE SSH PASSWORD LOGIN
    # ==========================================================

    echo "Configuring SSH..."

    sed -i \
      's/^[[:space:]]*PasswordAuthentication[[:space:]].*/# &/' \
      /etc/ssh/sshd_config

    sed -i \
      's/^[[:space:]]*PermitRootLogin[[:space:]].*/# &/' \
      /etc/ssh/sshd_config


    # Add our settings
    sed -i '1i PasswordAuthentication yes' /etc/ssh/sshd_config
    sed -i '1i PermitRootLogin yes' /etc/ssh/sshd_config


    # Check SSH drop-in configuration
    if [ -d /etc/ssh/sshd_config.d ]; then

      grep -Rl '^PasswordAuthentication no' \
        /etc/ssh/sshd_config.d 2>/dev/null | \
        xargs -r sed -i \
        's/^PasswordAuthentication no/PasswordAuthentication yes/'

    fi


    # Validate SSH configuration
    sshd -t

    # Restart SSH
    systemctl restart sshd


    # ==========================================================
    # 5. POSTGRESQL LAB STORAGE
    # ==========================================================

    echo
    echo "======================================"
    echo " Creating PostgreSQL Lab Filesystems "
    echo "======================================"


    #
    # Storage layout
    #
    # PostgreSQL DATA
    #   Default location:
    #   /var/lib/pgsql/18/data
    #
    # /pgwal      = 10 GB
    # /pgarchive  = 10 GB
    # /pgtabs     = 20 GB
    # /pgbackup   = 20 GB
    #


    # Directory for filesystem image files
    mkdir -p /var/lib/pgdisks


    # ==========================================================
    # 6. CREATE /pgwal FILESYSTEM
    # ==========================================================

    if [ ! -f /var/lib/pgdisks/pgwal.img ]; then

      echo "Creating /pgwal filesystem..."

      truncate -s 10G /var/lib/pgdisks/pgwal.img

      mkfs.xfs -f /var/lib/pgdisks/pgwal.img

    fi


    # ==========================================================
    # 7. CREATE /pgarchive FILESYSTEM
    # ==========================================================

    if [ ! -f /var/lib/pgdisks/pgarchive.img ]; then

      echo "Creating /pgarchive filesystem..."

      truncate -s 10G /var/lib/pgdisks/pgarchive.img

      mkfs.xfs -f /var/lib/pgdisks/pgarchive.img

    fi


    # ==========================================================
    # 8. CREATE /pgtabs FILESYSTEM
    # ==========================================================

    if [ ! -f /var/lib/pgdisks/pgtabs.img ]; then

      echo "Creating /pgtabs filesystem..."

      truncate -s 20G /var/lib/pgdisks/pgtabs.img

      mkfs.xfs -f /var/lib/pgdisks/pgtabs.img

    fi


    # ==========================================================
    # 9. CREATE /pgbackup FILESYSTEM
    # ==========================================================

    if [ ! -f /var/lib/pgdisks/pgbackup.img ]; then

      echo "Creating /pgbackup filesystem..."

      truncate -s 20G /var/lib/pgdisks/pgbackup.img

      mkfs.xfs -f /var/lib/pgdisks/pgbackup.img

    fi


    # ==========================================================
    # 10. CREATE MOUNT POINTS
    # ==========================================================

    mkdir -p /pgwal

    mkdir -p /pgarchive

    mkdir -p /pgtabs

    mkdir -p /pgbackup


    # ==========================================================
    # 11. CONFIGURE /etc/fstab
    # ==========================================================

    echo "Configuring /etc/fstab..."


    grep -q '/var/lib/pgdisks/pgwal.img' /etc/fstab || \
      echo '/var/lib/pgdisks/pgwal.img /pgwal xfs loop,nofail 0 0' \
      >> /etc/fstab


    grep -q '/var/lib/pgdisks/pgarchive.img' /etc/fstab || \
      echo '/var/lib/pgdisks/pgarchive.img /pgarchive xfs loop,nofail 0 0' \
      >> /etc/fstab


    grep -q '/var/lib/pgdisks/pgtabs.img' /etc/fstab || \
      echo '/var/lib/pgdisks/pgtabs.img /pgtabs xfs loop,nofail 0 0' \
      >> /etc/fstab


    grep -q '/var/lib/pgdisks/pgbackup.img' /etc/fstab || \
      echo '/var/lib/pgdisks/pgbackup.img /pgbackup xfs loop,nofail 0 0' \
      >> /etc/fstab


    # Mount all filesystems
    mount -a


    # ==========================================================
    # 12. CREATE POSTGRES OS USER
    # ==========================================================

    # PostgreSQL RPM will normally create the postgres user.
    # We create it here first because filesystem ownership
    # needs to be configured before PostgreSQL installation.

    if ! id postgres >/dev/null 2>&1; then

      useradd postgres

    fi


    # ==========================================================
    # 13. FILESYSTEM OWNERSHIP
    # ==========================================================

    chown postgres:postgres /pgwal
    chown postgres:postgres /pgarchive
    chown postgres:postgres /pgtabs
    chown postgres:postgres /pgbackup


    chmod 700 /pgwal
    chmod 700 /pgarchive
    chmod 700 /pgtabs
    chmod 700 /pgbackup


    # ==========================================================
    # 14. INSTALL POSTGRESQL REPOSITORY
    # ==========================================================

    echo
    echo "======================================"
    echo " Installing PostgreSQL Repository "
    echo "======================================"


    dnf install -y \
      https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm


    # ==========================================================
    # 15. DISABLE ROCKY DEFAULT POSTGRESQL MODULE
    # ==========================================================

    echo "Disabling default PostgreSQL module..."

    dnf -qy module disable postgresql


    # ==========================================================
    # 16. INSTALL POSTGRESQL 18 SERVER
    # ==========================================================

    echo
    echo "======================================"
    echo " Installing PostgreSQL 18 "
    echo "======================================"


    dnf install -y postgresql18-server
	 dnf install -y postgresql18-contrib


    # ==========================================================
    # 17. INITIALIZE POSTGRESQL DATABASE
    # ==========================================================

    echo
    echo "======================================"
    echo " Initializing PostgreSQL 18 "
    echo "======================================"


    /usr/pgsql-18/bin/postgresql-18-setup initdb


    # ==========================================================
    # 18. ENABLE POSTGRESQL SERVICE
    # ==========================================================

    echo "Enabling PostgreSQL service..."

    systemctl enable postgresql-18


    # ==========================================================
    # 19. START POSTGRESQL SERVICE
    # ==========================================================

    echo "Starting PostgreSQL service..."

    systemctl start postgresql-18


    # ==========================================================
    # 20. DISPLAY FILESYSTEMS
    # ==========================================================

    echo
    echo "======================================"
    echo " PostgreSQL Lab Filesystems "
    echo "======================================"


    df -hT | grep -E 'pgwal|pgarchive|pgtabs|pgbackup'


    # ==========================================================
    # 21. DISPLAY NETWORK
    # ==========================================================

    echo
    echo "======================================"
    echo " Network Configuration "
    echo "======================================"


    ip -4 addr


    # ==========================================================
    # 22. DISPLAY SSH CONFIGURATION
    # ==========================================================

    echo
    echo "======================================"
    echo " SSH Configuration "
    echo "======================================"


    sshd -T | grep -E \
      'passwordauthentication|permitrootlogin'


    # ==========================================================
    # 23. DISPLAY POSTGRESQL VERSION
    # ==========================================================

    echo
    echo "======================================"
    echo " PostgreSQL Version "
    echo "======================================"


    /usr/pgsql-18/bin/postgres --version


    # ==========================================================
    # 24. DISPLAY POSTGRESQL DATA DIRECTORY
    # ==========================================================

    echo
    echo "======================================"
    echo " PostgreSQL Data Directory "
    echo "======================================"


    sudo -u postgres \
      /usr/pgsql-18/bin/psql \
      -c "SHOW data_directory;"


    # ==========================================================
    # 25. DISPLAY POSTGRESQL PORT
    # ==========================================================

    echo
    echo "======================================"
    echo " PostgreSQL Port "
    echo "======================================"


    sudo -u postgres \
      /usr/pgsql-18/bin/psql \
      -c "SHOW port;"


    # ==========================================================
    # 26. DISPLAY POSTGRESQL SERVICE STATUS
    # ==========================================================

    echo
    echo "======================================"
    echo " PostgreSQL Service Status "
    echo "======================================"


    systemctl --no-pager status postgresql-18 || true


    # ==========================================================
    # FINAL
    # ==========================================================

    echo
    echo "======================================"
    echo " PostgreSQL Lab VM Ready "
    echo "======================================"

    echo
    echo "Hostname      : postgres-vm"
    echo "Private IP    : 192.168.136.10"
    echo "PostgreSQL    : 18"
    echo "Port          : 5432"
    echo "PGDATA        : /var/lib/pgsql/18/data"
    echo
    echo "/pgwal        : 10 GB"
    echo "/pgarchive    : 10 GB"
    echo "/pgtabs       : 20 GB"
    echo "/pgbackup     : 20 GB"
    echo
    echo "Root password : welcome1"
    echo
    echo "======================================"

  SHELL

end