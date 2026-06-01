# OpenShift Virtualization Operator

**Version: 0.1**

## Usage

This is to deploy [OpenShift Virtualization](https://docs.openshift.com/container-platform/4.5/virt/about-virt.html) into an OpenShift cluster. This operator
enables you to run virtual machines in an OpenShift cluster managed by the OpenShift control plane.

Note that this operator is only supported on bare metal deployments. For workshop or nested environments, use the **emulation** overlay (`KVM_EMULATION=true`).

### Operator (OLM)

```bash
oc apply -k virtualization-operator/overlays/emulation
oc wait --for=jsonpath='{.status.phase}'=Succeeded csv -n openshift-cnv -l operators.coreos.com/kubevirt-hyperconverged.openshift-cnv --timeout=1200s
```

### Instance (HyperConverged CR)

After the CSV is Succeeded, deploy the HyperConverged instance:

```bash
oc apply -k virtualization-operator/instance/base
```

Used by [openshift-agent-install](https://github.com/tosin2013/openshift-agent-install) `./hack/install-cnv-hub.sh`.