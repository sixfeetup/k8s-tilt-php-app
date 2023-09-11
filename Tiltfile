print(
    """
-----------------------------------------------------------------
✨ Hello Tilt! This appears in the (Tiltfile) pane whenever Tilt
   evaluates this file.
-----------------------------------------------------------------
""".strip()
)

load("ext://syncback", "syncback")

docker_build(
    "myproject-laravel-image",
    context="my-project",
    build_args={"DEVEL": "yes"},
    live_update=[
        sync("./my-project", "/app"),
    ],
)

k8s_yaml(
    kustomize("./k8s/dev/")
)

syncback(
    "backend-sync",
    "deploy/laravel",
    "/app/",
    target_dir="./my-project",
    rsync_path='/app/bin/rsync.tilt',
)

k8s_resource(workload='laravel', port_forwards=8000)
k8s_resource(workload='mariadb', port_forwards=3306)
