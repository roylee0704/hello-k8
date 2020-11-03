init:
	kubectl create ns roy-nginx

apply: # can be used as update too
	kubectl -n roy-nginx apply -f ./run-my-nginx.yaml

validate:
	kubectl -n roy-nginx get pods -o wide
	# get pod virtual ip
	kubectl -n roy-nginx get pods -o yaml | grep 'podIP:'

start: init apply validate

clean:
	kubectl -n roy-nginx delete -f run-my-nginx.yaml 