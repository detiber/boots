# Load the extension for live updating
load('ext://restart_process', 'docker_build_with_restart')

local_resource(
    'boots-build',
    'CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o build/boots ./',
    deps=[
        'go.mod',
        'go.sum',
        'main.go',
        'http.go',
        'tftp.go',
        'dhcp.go',
        'conf',
        'dhcp',
        'files',
        'httplog',
        'installers',
        'ipxe',
        'job',
        'metrics',
        'packet',
        'syslog',
        'tftp',
    ]
)

docker_build_with_restart(
    'quay.io/tinkerbell/boots',
    '.',
    dockerfile_contents="""
FROM gcr.io/distroless/base:debug as debug
WORKDIR /
COPY build/boots /boots
ENTRYPOINT ["/boots"]
""",
    only=[
        './build/boots',
    ],
    target='debug',
    live_update=[
        sync('./build/boots', '/boots')
    ],
    entrypoint=[
        '/boots',
        '-dhcp-addr=0.0.0.0:67',
        '-tftp-addr=0.0.0.0:69',
        '-http-addr=0.0.0.0:80',
        '-syslog-addr=0.0.0.0:514',
        '-log-level=DEBUG'
    ]
)

k8s_yaml('manifests/boots.yaml')
k8s_resource(
    workload='boots',
    resource_deps=[
        'tink-server',
    ]
)
