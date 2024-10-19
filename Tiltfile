    # version_settings() enforces a minimum Tilt version
# https://docs.tilt.dev/api.html#api.version_settings
version_settings(constraint='>=0.22.2')

# deploy DDB Helm Chart
load('ext://helm_resource', 'helm_resource', 'helm_repo')
helm_repo('keyporttech', 'https://keyporttech.github.io/helm-charts', labels=['DynamoDB'])
helm_resource('helm-ddb', 'keyporttech/dynamodb', resource_deps=['keyporttech'], port_forwards=['8000:8000', '8001:8001'], labels=['DynamoDB'])

#Install Kafka
#create namespaces, zk, and kafka brokers
k8s_yaml('./config/kafka/00-namespace.yaml')
k8s_yaml('./config/kafka/01-zookeeper.yaml')
k8s_yaml('./config/kafka/02-kafka.yaml')

#add labels for categorization in the tit UI and and port forwards so we can access
#the cluster from outside the cluster for testing (e.g., using kcat on local machine)
k8s_resource(workload='zookeeper', labels=['Kafka_Cluster'])
k8s_resource(workload='kafka-broker', port_forwards=9092, resource_deps=['zookeeper'], labels=['Kafka_Cluster'])


#configure go API service
docker_build('api-server', './api', dockerfile='./api/Dockerfile')
k8s_yaml('./api/deploy.yaml')
k8s_resource(workload='api-server', port_forwards=8080, labels=['api-server'], resource_deps=['kafka-broker'])


#configure go Kafka consumer service
docker_build('kconsumer-server', './kconsumer', dockerfile='./kconsumer/Dockerfile')
k8s_yaml('./kconsumer/deploy.yaml')
k8s_resource(workload='kconsumer-server', labels=['kconsumer-server'], resource_deps=['helm-ddb', 'kafka-broker'])

# config.main_path is the absolute path to the Tiltfile being run
# there are many Tilt-specific built-ins for manipulating paths, environment variables, parsing JSON/YAML, and more!
# https://docs.tilt.dev/api.html#api.config.main_path
tiltfile_path = config.main_path

# print writes messages to the (Tiltfile) log in the Tilt UI
# the Tiltfile language is Starlark, a simplified Python dialect, which includes many useful built-ins
# config.tilt_subcommand makes it possible to only run logic during `tilt up` or `tilt down`
# https://github.com/bazelbuild/starlark/blob/master/spec.md#print
# https://docs.tilt.dev/api.html#api.config.tilt_subcommand
if config.tilt_subcommand == 'up':
    print("""
    \033[32m\033[32mHello World from tilt-avatars!\033[0m

    If this is your first time using Tilt and you'd like some guidance, we've got a tutorial to accompany this project:
    https://docs.tilt.dev/tutorial

    If you're feeling particularly adventurous, try opening `{tiltfile}` in an editor and making some changes while Tilt is running.
    What happens if you intentionally introduce a syntax error? Can you fix it?
    """.format(tiltfile=tiltfile_path))