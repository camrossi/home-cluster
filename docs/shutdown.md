kcn argocd
k scale statefulset --replicas 0 --all
k scale deployment --replicas 0 --all

kcn webtop
k scale deployment --replicas 0 --all

kcn obsidian-sync
k scale statefulset --replicas 0 --all

kcn netbox
k scale statefulset --replicas 0 --all
k scale deployment --replicas 0 --all

kcn jellyfin
k scale deployment --replicas 0 --all

kcn spliit
k scale statefulset --replicas 0 --all
k scale deployment --replicas 0 --all

kcn mealie
k scale statefulset --replicas 0 --all
k scale deployment --replicas 0 --all

kcn typo-api
k scale deployment --replicas 0 --all

kcn manyfold
k scale deployment --replicas 0 --all
k scale statefulset --replicas 0 --all

kcn gitea
k scale deployment --replicas 0 --all

kcn uptime-kuma
k scale deployment --replicas 0 --all

kcn registry
k scale deployment --replicas 0 --all

kcn immich
k scale deployment --replicas 0 --all
kubectl annotate cluster  immich-database --overwrite cnpg.io/hibernation=on

kcn kube-prometheus-stack
k scale statefulset --replicas 0 --all
k scale deployment --replicas 0 --all

kcn ocis
k scale deployment --replicas 0 --all

kcn navidrome
k scale deployment --replicas 0 --all
k scale statefulset --replicas 0 --all

kcn ftwr
k scale deployment --replicas 0 --all
k scale statefulset --replicas 0 --all

kcn home-assistant
k scale deployment --replicas 0 --all
k scale statefulset --replicas 0 --all

kcn authentik
k scale deployment --replicas 0 --all
k scale statefulset --replicas 0 --all

kcn cloudnative-pg
k scale deployment --replicas 0 --all

kcn pihole
k scale deployment --replicas 0 --all
k scale statefulset --replicas 0 --all

kcn cert-manager
k scale deployment --replicas 0 --all

k cordon rpi4-4 rpi4-3 rpi4-2 rpi4-1 rk1-4 rk1-1 rk1-2 rk1-3

Shutdown workers:
talosctl -n 192.168.0.10,192.168.0.7,192.168.0.9,192.168.0.6,192.168.0.8 shutdown

Shutdown cp2 and cp3:
talosctl -n 192.168.0.5,192.168.0.4 shutdown