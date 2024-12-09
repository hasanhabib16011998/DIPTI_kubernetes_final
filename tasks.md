
# task 1
- make a index.html file
- make a docker file for this index file
	- base image will be nginx
	- this index file should be served via nginx server
-	make a kubernetes deployment file for that docker image
-	expose the deployment via nodeport

upload manifest files,dockerfile,src code to the github


# task 2
- make a kubernetes deployment file 
- this deployment will have 2 containers defined
- both container will refer a emptydir volume, mounting /mount volume each
- container 1,2 both will have a alpine image
- write a custom command in container 1 so that it simply creates a file every minute in /mount volume
	- ex. log1.txt, log2.txt, log3.txt .. ... .. etc
- write a custom command in container 2 so that it simply creates a file every minute in /mount volume
	- ex. file1.txt, file2.txt, file3.txt .. ... ..  etc

upload manifest files on github

Hi
