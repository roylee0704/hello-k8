init:
	kubectl create ns rbac-test

apply:
	kubectl create deploy nginx --image=nginx -n rbac-test

destroy:
	kubectl delete ns rbac-test

status:
	kubectl get all -n rbac-test