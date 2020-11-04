init:
	kubectl create ns roy-nginx

apply: # can be used as update too
	kubectl -n roy-nginx apply -f ./run-my-nginx.yaml

validate:
	kubectl -n roy-nginx get pods -o wide
	# get pod virtual ip
	kubectl -n roy-nginx get pods -o yaml | grep 'podIP:'

start: init apply validate

destroy:
	kubectl -n roy-nginx delete -f run-my-nginx.yaml


patch_svc_loadbalancer:
	kubectl -n roy-nginx patch svc roy-nginx -p '{"spec": {"type": "LoadBalancer"}}'

# note: you can't patch from loadbalancer to clusterip. However the reverse is possible.
patch_svc_clusterip:
	kubectl -n roy-nginx patch svc roy-nginx -p '{"spec": {"type": "ClusterIP"}}'