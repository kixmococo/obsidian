sudo tee /etc/udev/rules.d/99-rtw89-no-d3cold.rules << 'EOF'
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10ec", ATTR{device}=="0x892b", ATTR{power/control}="on"
EOF
sudo udevadm control --reload-rules