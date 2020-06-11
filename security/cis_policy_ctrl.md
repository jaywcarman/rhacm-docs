# CIS policy controller

You can use the Center for Internet Security (CIS) policy controller to receive notifications about non-compliant clusters. **Note**: THe CIS policy controller is only supported on OpenShift Container Platform 3.11.

The CIS policy controller monitors the nodes in a cluster for compliance against CIS Kubernetes benchmark checks. The CIS policies that list the rules to exclude can be applied to the managed clusters. The controller checks the cluster for any violations that are not in the exclude list.

The controller uses the Aqua security [`kube-bench` tool](https://github.com/aquasecurity/kube-bench) to check the master and worker nodes in the managed cluster for compliance. For information, see [Aqua security](https://www.aquasec.com/).

When a managed cluster is non-compliant, the CIS controller assigns a risk score. Each CIS rule that fails the check is assigned a score. The risk score that is assigned to the non-compliant cluster is the maximum of all the scores that are assigned to failed checks.

The risk score is on a scale of 1 to 10.
*  If the score is less than 4, then the risk category is `low`.
*  If the score is greater than or equal to 4, and less than or equal to 7, then the risk category is `medium`.
*  If the score is greater than 7, then the risk category is `high`.

## CIS policy controller components

The CIS policy controller consists of the following four components.

* `cis-controller-minio`: The `cis-controller-minio` object store is used to store the artifacts that are collected by the CIS crawler that runs on all the master and worker nodes. The results from running the Aqua security `kube-bench` tool are also stored in the CIS Minio object store.

* `cis-crawler`: The `cis-crawler` collects information about Kubernetes processes, binary files, and configuration files and stores them in the Minio object store. The crawler runs on the master and worker nodes. The crawler runs every 24 hours.

* `drishti-cis`: The `drishti-cis` component runs the Aqua security `kube-bench` tool against the artifacts that are collected by the `cis-crawler` and stores the results in `cis-controller-minio` object store. The artifacts are scanned every 24 hours.

* `cis-controller`: The `cis-controller` scans the `cis-controller-minio` object store for results that are generated by the Aqua security `kube-bench` tool and updates the CIS policy status. The controller scans the results every 24 hours or whenever the policy is updated.

## CIS policy YAML structure

```yaml
apiVersion: policy.mcm.ibm.com/v1alpha1
kind: Policy
metadata:
  name:
  namespace:
  annotations:
    policy.mcm.ibm.com/categories:
    policy.mcm.ibm.com/controls:
    policy.mcm.ibm.com/standards:
    seed-generation:
  finalizers:
    - finalizer.policies.ibm.com
    - propagator.finalizer.mcm.ibm.com
  generation:
  resourceVersion:
spec:
  complianceType:
  namespaces:
    exclude:
    include:
  policy-templates:
    - objectDefinition:
        apiVersion: policies.ibm.com/v1alpha1
        kind: CisPolicy
        metadata:
          name:
          labels:
            controller-tools.k8s.io:
            pci-category: system-integrity
        spec:
          cisSpecVersion: 1.4
          kubernetesCisPolicy:
            masterNodeExcludeRules:
            workerNodeExcludeRules:
          severity:
  remediationAction:
  disabled:
```

### CIS policy YAML table

|Field|Description|
|-- | -- |
| apiVersion | Required. Set the value to `policy.mcm.ibm.com/v1alpha1`. |
| kind | Required. Set the value to `Policy` to indicate the type of policy. |
| metadata.name | Required. The name for identifying the policy resource. |
| metadata.namespace | Required. The namespaces within the managed cluster where the policy is created. |
| metadata.annotations | Optional. Used to specify a set of security details that describes the set of standards the policy is trying to validate. **Note**: You can view policy violations based on the standards and categories that you define for your policy on the _Policies_ page, from the console. |
| annotations.policy.mcm.ibm.com/standards | Optional. The name or names of security standards the policy is related to. For example, National Institute of Standards and Technology (NIST) and Payment Card Industry (PCI).|
| annotations.policy.mcm.ibm.com/categories | A security control category represent specific requirements for one or more standards. For example, a System and Information Integrity category might indicate that your policy contains a data transfer protocol to protect personal information, as required by the HIPAA and PCI standards. |
| annotations.policy.mcm.ibm.com/controls | The name of the security control that is being checked. For example, Center of Internet Security (CIS) and certificate policy controller.|
| spec | Required. Specifications of which CIS policies to monitor and refresh. |
| spec.complianceType | Required. Set the value to `"musthave"`|
| spec.namespace | Required. The namespaces within the hub cluster that the policy is applied to. Enter parameter values for `include`, which are the namespaces you want to apply to the policy to. The `exclude` parameter specifies the namespaces you explicitly do not want to apply the policy to. **Note**: A namespace that is specified in the object template of a policy controller overrides the namespace in the corresponding parent policy.|
| spec.policy-template | Optional. Used to create one or more policies for third party or external security controls. |
| policy-template.objectDefinition | Optional. |
| obejctDefinition.labels| Optional. |
| objectDefinition.cisSpecVersion | Required. |
| objectDefintion.kubernetesCisPolicy| Required. Refer to OpenShift Container Platform CIS rules when you create CIS policies. Enter parameter values for the `masterNodeExcludeRules` and `workerNodeExcludeRules`. See [CIS rules specifications](cis_policy_rules.md) for a list of the rules. |
| objectDefinition.severity | Required. Identify the details for a specific severity level. |
| disabled | Required. Set the value to `true` or `false`. The `disabled` parameter provides the ability to enable and disable your policies. CIS policy controller is disabled by default.|
| remediationAction | Required. Specifies the remediation of your policy. Enter `inform`. |
{: caption="Table 1. Required and optional definition fields" caption-side="top"}


Learn to create and manage your CIS policy, see [Managing CIS policies](create_cis_pol.md). Refer to [Policy controllers](policy_controllers.md) for more topics.