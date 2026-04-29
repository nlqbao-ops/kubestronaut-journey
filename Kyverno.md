### curriculum 

#### fundatmentals
- Kyverno policies and rules
- yaml manifests
- admission controllers
- oci images

#### installation, config, and upgrades
- helm-based installation and config
- kyverno CRDs
- controller config with flags 
- config kyverno RBAC, roles, and permissions 
- HA installations 
- upgrading kyverno 

#### kyverno cli
- apply 
- test 
- jp
- install kyverno cli

#### applying policies 
- apply policy in cluster
- resource selection 
- common policy settings for kyverno rules 

#### writing policies 
- validation rules 
- preconditions 
- background scans 
- mutation rules 
- generation rules 
- verifyImage rules 
- variables and API call in policies 
- JSON patches 
- autogen rules
- cleanup policies 
- common expression language 

#### policy mgmt 
- policy report 
- policyExceptions 
- kyverno metrics 


### resources 

Kyverno Playground
https://playground.kyverno.io/

### course 

### KodeKloud KCA course mapping

credit 
Course: Prep Course - Kyverno Certified Associate (KCA) Certification  
https://learn.kodekloud.com/courses/kyverno-certified-associate

Course includes:
- 13 modules
- 119 lessons
- 29 labs
- 11 quizzes
- 2 mock exams
- 11.22 hours total
- Instructor: Mariam Fahmy, Kyverno maintainer

#### course to documentation mapping

| KodeKloud module | Main topics | Kyverno documentation |
|---|---|---|
| Course Introduction | Course setup and community | N/A |
| Kyverno Introduction | Overview, architecture, installation, policy structure | https://kyverno.io/docs/installation/ <br> https://kyverno.io/docs/policy-types/overview/ |
| Resource Filters | match, exclude, any/all, JMESPath, preconditions | https://kyverno.io/docs/policy-types/cluster-policy/match-exclude/ <br> https://kyverno.io/docs/policy-types/cluster-policy/preconditions/ |
| Validate Rules | validation, failure actions, patterns, anchors, deny rules, foreach, Pod Security, CEL, autogen | https://kyverno.io/docs/policy-types/validating-policy/ <br> https://kyverno.io/docs/guides/pod-security/ |
| Mutate Rules | JSONPatch, strategic merge patch, conditional anchors, mutate existing resources, foreach | https://kyverno.io/docs/policy-types/mutating-policy/ |
| Generate Rules | data source, clone source, clone list, generate existing, synchronization behavior | https://kyverno.io/docs/policy-types/generating-policy/ |
| External Data Sources | ConfigMaps, API calls, global context, image registry variables | https://kyverno.io/docs/policy-types/cluster-policy/external-data-sources/ |
| ImageVerify Rules | image signing, Notary signatures, attestations, supply chain security | https://kyverno.io/docs/policy-types/image-validating-policy/ |
| Policy Exceptions | PolicyException resources, Pod Security exemptions | https://kyverno.io/docs/guides/exceptions/ |
| Cleanup Policies | cleanup policies, cleanup labels, scheduled deletion | https://kyverno.io/docs/policy-types/deleting-policy/ <br> https://kyverno.io/docs/policy-types/cleanup-policy/ |
| Reporting | PolicyReport schema, admission reports, background scan reports | https://kyverno.io/docs/guides/reports/ |
| Kyverno CLI | kyverno apply, kyverno test | https://kyverno.io/docs/kyverno-cli/ <br> https://kyverno.io/docs/kyverno-cli/reference/kyverno_apply/ <br> https://kyverno.io/docs/kyverno-cli/reference/kyverno_test/ |
| Mock Exams | Exam readiness | Use after finishing course labs and docs review |

#### extra documentation to study outside the course

The KodeKloud course looks strong for hands-on policy writing, but review these official docs separately:

- Installation and configuration: https://kyverno.io/docs/installation/
- Customization and controller flags: https://kyverno.io/docs/installation/customization/
- High availability: https://kyverno.io/docs/guides/high-availability/
- Upgrading Kyverno: https://kyverno.io/docs/installation/upgrading/
- CRDs: https://kyverno.io/docs/crds/
- Admission controllers: https://kyverno.io/docs/guides/admission-controllers/
- Monitoring: https://kyverno.io/docs/guides/monitoring/
- Metrics reference: https://kyverno.io/docs/reference/metrics/
- kyverno jp: https://kyverno.io/docs/kyverno-cli/reference/kyverno_jp/
- Policy library: https://kyverno.io/policies/
- Kyverno Playground: https://playground.kyverno.io/

#### study notes

- Use KodeKloud as the main study path for policy authoring.
- Use official docs to fill gaps around installation, RBAC, CRDs, HA, upgrades, metrics, and kyverno jp.
- Practice both classic policy syntax and newer CEL-based policy types.
- Classic syntax: Policy, ClusterPolicy, validate, mutate, generate, verifyImages.
- Newer policy types: ValidatingPolicy, MutatingPolicy, GeneratingPolicy, DeletingPolicy, ImageValidatingPolicy.
- The exam may still rely heavily on classic Policy and ClusterPolicy examples, even though newer docs emphasize CEL-based policy types.