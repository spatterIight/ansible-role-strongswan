# Molecule Testing

## Installation

```bash
python3 -m venv venv
source venv/bin/activate
pip3 install -r requirements.yml
```

# Execution

Ubuntu 24.04:
```bash
MOLECULE_DISTRO=ubuntu2404 molecule --scenario-name stroke test

MOLECULE_DISTRO=ubuntu2404 molecule --scenario-name vici test
```

Ubuntu 22.04:
```bash
MOLECULE_DISTRO=ubuntu2204 molecule --scenario-name stroke test

MOLECULE_DISTRO=ubuntu2204 molecule --scenario-name vici test
```
