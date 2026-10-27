name: Build TWRP Infinix Hot 30i

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-20.04
    permissions:
      contents: write
    
    steps:
    - name: Checkout
      uses: actions/checkout@v3

    - name: Membersihkan Lingkungan & Install Dependencies
      run: |
        sudo apt-get update
        sudo apt-get install -y gperf gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig python3 rsync
        docker rmi $(docker images -q)
        sudo rm -rf /usr/share/dotnet /etc/mysql /etc/php /usr/local/lib/android
        sudo rm -rf /opt/ghc /usr/local/share/boost /var/lib/docker

    - name: Setup Repo Tool
      run: |
        mkdir ~/bin
        curl https://googleapis.com > ~/bin/repo
        chmod a+x ~/bin/repo
        sudo ln -sf ~/bin/repo /usr/bin/repo

    - name: Sinkronisasi Source Code TWRP
      run: |
        mkdir twrp
        cd twrp
        git config --global user.name "Android Builder"
        git config --global user.email "builder@android.local"
        repo init --depth=1 -u https://github.com -b twrp-12.1
        repo sync -c -j$(nproc --all) --force-sync --no-clone-bundle --no-tags

    - name: Kloning Device Tree Infinix Hot 30i
      run: |
        cd twrp
        # Ganti URL di bawah dengan repositori device tree hasil buatan twrpdtgen Anda
        git clone https://github.com device/infinix/X669C

    - name: Memulai Proses Build TWRP
      run: |
        cd twrp
        export ALLOW_MISSING_DEPENDENCIES=true
        source build/envsetup.sh
        lunch omni_X669C-eng
        mka recoveryimage -j$(nproc --all)

    - name: Unggah Hasil Build (Artifact)
      uses: actions/upload-artifact@v3
      with:
        name: TWRP-Infinix-Hot30i
        path: twrp/out/target/product/X669C/recovery.img
        
