# script_install_node
#!/bin/bash

set -e  # Exit script if any command fails

echo "🔄 Updating the system..."
# Install needrestart if not installed
sudo apt-get install -y needrestart

# Disable needrestart notifications
sudo sed -i 's/#\$nrconf{restart} =.*/$nrconf{restart} = "a";/' /etc/needrestart/needrestart.conf

export NEEDRESTART_MODE=a  # Skip restart service prompts
sudo DEBIAN_FRONTEND=noninteractive apt-get update
sudo DEBIAN_FRONTEND=noninteractive apt-get -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" upgrade -y

echo "📦 Installing necessary packages..."
sudo apt-get install -y screen curl libssl-dev pkg-config build-essential git-all protobuf-compiler unzip

# Install Protobuf (protoc)
PROTOC_VERSION=29.3
PROTOC_ZIP="protoc-${PROTOC_VERSION}-linux-x86_64.zip"

echo "⬇️ Downloading and installing protoc ${PROTOC_VERSION}..."
cd /usr/local
sudo wget -q https://github.com/protocolbuffers/protobuf/releases/download/v${PROTOC_VERSION}/${PROTOC_ZIP}
sudo unzip -o ${PROTOC_ZIP}
sudo chmod +x /usr/local/bin/protoc

# Check if protoc is installed
if ! command -v protoc &> /dev/null; then
    echo "❌ Error: protoc installation failed, please check!"
    exit 1
else
    echo "✅ Protoc installed successfully: $(protoc --version)"
fi

# Install Rust
echo "🦀 Installing Rust..."
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"

# Add Rust target for riscv32i-unknown-none-elf
echo "🔧 Adding target riscv32i-unknown-none-elf for Rust..."
cd /root/.nexus/network-api/clients/cli && rustup target add riscv32i-unknown-none-elf

echo "✅ Installation complete, run 'screen -S nexus' to start"
