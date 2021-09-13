# Install Quay Operator
## Notes
    - S3 storage must be available either through OCS noobaa or other services.
    - Mirroring just a single version of OCP will require approximately 12GB of storage
    - Mirroring Operators can take up to 365GB+ of storage (depending on how many operators selected) just for the Red Hat operators. Reason being that at present (Sep. '21) you cannot just mirror the latest release of an operator.
    - Therefore the default 50GB PV for noobaa will not be sufficient and needs to be expanded. 

1. Install the quay operator from the operator hub

