SOURCE_IMAGE = os.getenv("SOURCE_IMAGE", default='harbor.vrabbi.cloud/guylab/spring-pet-clinic-kotlin-app-source')
LOCAL_PATH = os.getenv("LOCAL_PATH", default='.')
NAMESPACE = os.getenv("NAMESPACE", default='apps')

local_resource(
  'extract-jar',
  gradlew + ' bootJar && ' +
  'unzip -o build/libs/example-0.0.1-SNAPSHOT.jar -d build/jar-staging && ' +
  'rsync --inplace --checksum -r build/jar-staging/ build/jar',
  deps=['src', 'build.gradle.kts'])

k8s_custom_deploy(
    'pet-clinic',
    apply_cmd="tanzu apps workload apply -f config/workload.yaml --update-strategy replace --debug --live-update" +
               " --local-path " + LOCAL_PATH +
               " --source-image " + SOURCE_IMAGE +
               " --namespace " + NAMESPACE +
               " --yes --output yaml",
    delete_cmd="tanzu apps workload delete -f config/workload.yaml --namespace " + NAMESPACE + " --yes",
    deps=['src', 'build.gradle.kts'],
    resource_deps = ['extract-jar'])
    container_selector='workload',
    live_update=[
      sync('./build/jar/BOOT-INF/lib', '/app/lib'),
      sync('./build/jar/META-INF', '/app/META-INF'),
      sync('./build/jar/BOOT-INF/classes', '/app'),
    ]
)

k8s_resource('pet-clinic', port_forwards=["8080:8080"],
            extra_pod_selectors=[{'carto.run/workload-name': 'pet-clinic', 'app.kubernetes.io/component': 'run'}])
