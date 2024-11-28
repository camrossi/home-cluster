1) Install the the controller
NAMESPACE="actions-runner"
helm install arc \
    --namespace "${NAMESPACE}" \
    --create-namespace \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller

2) 
NAMESPACE="actions-runner"
GITHUB_CONFIG_URL="https://github.com/camsab"
GITHUB_PAT="<YOUR KEY>"
helm upgrade --install home-runners \
    --namespace "${NAMESPACE}" \
    --create-namespace \
    --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
    --set githubConfigSecret.github_token="${GITHUB_PAT}" \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set \
    -f values.yaml

Upgrade: Uninstall and re install 


3) To be able to run build kit I need to be able to create deployments it seems so need to add this
 kubectl edit role home-runners-gha-rs-kube-mode
 rules:
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - get
  - list
  - create
  - delete
- apiGroups:
  - ""
  resources:
  - configmaps
  verbs:
  - get
  - list
  - create
  - delete

4) Depenign what you are bulding you will need more disk space, to build the talos images the scratch partition cab be very big 5G+ and that will not work well with eMMC much better to just boot from NVME
5) Talos build process requires:
  - root proviledges to be able to use loop devices and '/dev' needs to be passed to the container
  - the /out partition needs to be shared between the buildX POD and the Runner pod so 
  To address both the above needs I have added a dedicated PVC and is then moutned via a `hook-extension`