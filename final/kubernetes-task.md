# task
- create a kubernetes configmap
	- where data is name = yourname

- create a deployment
	- 1 replica
	- image: alpine
	- add custom command : 
		- run a infinite loop
		- echo the name variable from the configmap
		- sleep 5 second
		- loop continues forever

