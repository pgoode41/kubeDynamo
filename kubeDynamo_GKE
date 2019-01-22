package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"os"
	"os/exec"
	"strings"
)

func main() {

	//Pull JSON And Store It In A Variable
	url := "https://swapi.co/api/people"
	req, _ := http.NewRequest("GET", url, nil)
	res, _ := http.DefaultClient.Do(req)
	defer res.Body.Close()
	body, _ := ioutil.ReadAll(res.Body)

	//Parse Through JSON
	var character Character
	json.Unmarshal(body, &character)
	people := character.Results

	//serverList := [...]string{"watson", "mooncake", "somerandom"}

	//Create the dockerAuto Directory Tree and Building It IF It Doesn't Exist.
	dockerAutoPath := "/opt/dockerAuto"
	dockerBaseImagePath := "/opt/dockerAuto/dockerBaseImage"
	dockerCustomImagePath_ROOT := "/opt/dockerAuto/dockerGenImages"

	//dockerApplicationPATH := "/opt/dockerAppliactions"
	if _, err := os.Stat(dockerAutoPath); os.IsNotExist(err) {
		os.Mkdir(dockerAutoPath, 0755)
		os.Chmod(dockerAutoPath, 0755)
		if _, err := os.Stat(dockerCustomImagePath_ROOT); os.IsNotExist(err) {
			os.Mkdir(dockerCustomImagePath_ROOT, 0755)
			os.Chmod(dockerCustomImagePath_ROOT, 0755)
		}
		if _, err := os.Stat(dockerBaseImagePath); os.IsNotExist(err) {
			os.Mkdir(dockerBaseImagePath, 0755)
			os.Chmod(dockerBaseImagePath, 0755)
		}

	} else if _, err := os.Stat(dockerCustomImagePath_ROOT); os.IsNotExist(err) {
		os.Mkdir(dockerCustomImagePath_ROOT, 0755)
		os.Chmod(dockerCustomImagePath_ROOT, 0755)
	} else if _, err := os.Stat(dockerBaseImagePath); os.IsNotExist(err) {
		os.Mkdir(dockerBaseImagePath, 0755)
		os.Chmod(dockerBaseImagePath, 0755)
	}

	//List Of Illegal Characters For Variables.(Allows Spaces)
	//Use formatedJsonNameNOSpaces On Variable Directly
	badCharacterList := []string{"(", ")", `\`, `\`, ",", ".", "!", "@", "#", "$", "%", "^", "*", "+", "=", "[", "]", ";",
		`:`, "<", ">", "{", "}", `\`, "?", "/", "|", "`", "~", "_", `"`}

	//Change To dockerBaseImagePath Directory.
	if err := os.Chdir(dockerBaseImagePath); err != nil {
		panic(err)
	}

	//Creating Base Dockerfile from Ubuntu Image and Customizing.
	baseImageDockerFile_Name := "Dockerfile"
	baseImageDockerFile, err := os.Create(baseImageDockerFile_Name)
	os.Chmod(baseImageDockerFile_Name, 0755)
	if err != nil {
		log.Fatal("Cannot create ", baseImageDockerFile_Name, err)
	}
	defer baseImageDockerFile.Close()
	//Creating Base Dockerfile from Ubuntu Image and Customizing.
	//Name Must Stay Dockerfile (docker syntax rule).
	fmt.Fprintln(baseImageDockerFile, "FROM ubuntu")
	fmt.Fprintln(baseImageDockerFile, "RUN apt update -y && apt upgrade -y")
	fmt.Fprintln(baseImageDockerFile, "RUN apt install golang-go -y")
	fmt.Fprintln(baseImageDockerFile, "RUN apt install curl -y")
	fmt.Fprintln(baseImageDockerFile, "RUN apt install nano -y")
	fmt.Fprintln(baseImageDockerFile, "ENV TESTVAR1='This is an ENV var from dockerfile!'")
	baseImageDockerFile.Close()
	//Changing to the generated base Image directory.
	if err := os.Chdir(dockerBaseImagePath); err != nil {
		panic(err)
	}

	//Creating a bash script that runs the docker commands to make the image.
	dockerBaseImageNAME := "dockerbase-image"
	dockerBaseImageVERSION := "v1"
	bashBaseImageBuildScript_Name := "bashBaseImageBuildScript.sh"
	bashBaseImageBuildScript, err := os.Create(bashBaseImageBuildScript_Name)
	os.Chmod(bashBaseImageBuildScript_Name, 0755)
	if err != nil {
		log.Fatal(bashBaseImageBuildScript_Name, err)
	}
	defer baseImageDockerFile.Close()
	//Creating a bash script that runs the docker commands to make the image.
	fmt.Fprintln(bashBaseImageBuildScript, "#!/bin/bash")
	fmt.Fprintln(bashBaseImageBuildScript, "docker build -t", dockerBaseImageNAME+":"+dockerBaseImageVERSION, ".")
	fmt.Fprintln(bashBaseImageBuildScript, "exit 0")
	bashBaseImageBuildScript.Close()
	//Executing a bash script that runs the docker commands to make the image.
	BASHScriptEXE(dockerBaseImagePath + "/bashBaseImageBuildScript.sh")

	//Change To /opt/dockerTesting/dockerGenImages Directory.
	if err := os.Chdir(dockerCustomImagePath_ROOT); err != nil {
		panic(err)
	}

	for i := 0; i < len(people); i++ {
		//Create Variables From JSON
		// Using Name Method From Custom Type
		jsonName := people[i].Name
		//Removing Illegal Characters From Variable
		for _, badCharacter := range badCharacterList {
			jsonName = strings.Replace(jsonName, badCharacter, "", -1)
		}
		//Stores Data That Doesn't Contain Illegal Characters, But Allows Spaces.
		serverNameWSpaces := strings.Title(jsonName)
		//Stores Data That Doesn't Contain Illegal Characters, But DOES NOT Allow Spaces.
		serverNameNOSpaces := strings.Replace(serverNameWSpaces, " ", "", -1)
		serverNameNOSpacesLower := strings.ToLower(serverNameNOSpaces)
		//##################################################################################################################
		//##################################################################################################################

		if err := os.Chdir(dockerCustomImagePath_ROOT); err != nil {
			panic(err)
		}
		//Creating The Root Directory Fot The Generated Custom Images
		dockerCustomImagePath_IMAGES := dockerCustomImagePath_ROOT + "/" + serverNameNOSpaces
		if _, err := os.Stat(dockerCustomImagePath_IMAGES); os.IsNotExist(err) {
			os.Mkdir(dockerCustomImagePath_IMAGES, 0755)
			os.Chmod(dockerCustomImagePath_IMAGES, 0755)
		}

		//Change To Directory Were Custom Images Will Be Stored.
		if err := os.Chdir(dockerCustomImagePath_IMAGES); err != nil {
			panic(err)
		}

		/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
		/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
		gcloudDnsZoneName := "fourtyeightpennies-zone"
		gcloudDomainName := "48pennies.io"
		gcloudProject := "micromdm-test-v1"
		clusterName := serverNameNOSpacesLower + "-cluster"
		//clusterNameSpace := serverNameNOSpacesLower + "-namespace"
		clusterZone := "us-central1-a"
		//replicaCount := "2"
		externalPort := "80"
		containerAppPort := "8080"
		containerHostName := serverNameNOSpacesLower
		/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
		/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
		//Creating Custon Dockerfile file from JSON DATA.
		//Name Must be Dockerfile (Docker syntax rule).
		//Creating Custon Dockerfile file from JSON DATA.
		//Name Must be Dockerfile (Docker syntax rule).

		customImageDockerFile_Name := "Dockerfile"
		customImageDockerFile, err := os.Create(customImageDockerFile_Name)
		os.Chmod(customImageDockerFile_Name, 0755)
		if err != nil {
			log.Fatal("Cannot create ", customImageDockerFile_Name, err)
		}
		defer customImageDockerFile.Close()
		//Creating Custom Docker File From Json Data.
		fmt.Fprintln(customImageDockerFile, "FROM", dockerBaseImageNAME+":"+dockerBaseImageVERSION)
		fmt.Fprintln(customImageDockerFile, `ENV MDMHOST_NAME='`+containerHostName+`'`)
		fmt.Fprintln(customImageDockerFile, `ENV MDMDOMAIN_NAME='`+gcloudDomainName+`'`)
		fmt.Fprintln(customImageDockerFile, `ENV ADMIN_EMAIL='pgoode41@gmail.com'`)
		fmt.Fprintln(customImageDockerFile, `ENV DEBIAN_FRONTEND=noninteractive`)
		fmt.Fprintln(customImageDockerFile, "RUN mkdir /GitRepo")
		fmt.Fprintln(customImageDockerFile, "WORKDIR /GitRepo")
		fmt.Fprintln(customImageDockerFile, "RUN apt update -y")
		fmt.Fprintln(customImageDockerFile, `RUN curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh`)
		fmt.Fprintln(customImageDockerFile, `RUN chmod 700 get_helm.sh`)
		fmt.Fprintln(customImageDockerFile, `RUN ./get_helm.sh`)
		fmt.Fprintln(customImageDockerFile, "RUN apt install git -y")
		fmt.Fprintln(customImageDockerFile, `RUN git clone https://github.com/pgoode41/IBM_Project.git`)
		fmt.Fprintln(customImageDockerFile, "RUN mkdir /TestLoads")
		fmt.Fprintln(customImageDockerFile, "RUN apt update -y")
		fmt.Fprintln(customImageDockerFile, "RUN apt upgrade -y")
		fmt.Fprintln(customImageDockerFile, "WORKDIR /TestLoads")
		fmt.Fprintln(customImageDockerFile, `CMD ["go", "run", "/GitRepo/IBM_Project/Test_Payloads/webserver_EnvCar.go"]`)
		customImageDockerFile.Close()

		dockerCustomImageNAME := serverNameNOSpacesLower + "-image"
		dockerCustomImageVERSION := "v1"

		bashCustomImageBuildScript_Name := "bashCustomImageBuildScript.sh"
		bashCustomImageBuildScript, err := os.Create(bashCustomImageBuildScript_Name)
		os.Chmod(bashCustomImageBuildScript_Name, 0755)
		if err != nil {
			log.Fatal("Cannot create", bashCustomImageBuildScript_Name, err)
		}
		defer bashCustomImageBuildScript.Close()
		//Creating a bash script that runs the docker commands to make the image.
		fmt.Fprintln(bashCustomImageBuildScript, "#!/bin/bash")
		fmt.Fprintln(bashCustomImageBuildScript, "docker build -t", dockerCustomImageNAME+":"+dockerCustomImageVERSION, ".")
		fmt.Fprintln(bashCustomImageBuildScript, "exit 0")
		bashCustomImageBuildScript.Close()
		//Executing a bash script that runs the docker commands to make the image.
		BASHScriptEXE(dockerCustomImagePath_IMAGES + "/" + bashCustomImageBuildScript_Name)

		/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
		/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
		//Building Public Application Cluster
		//Configuring Host Names, creating A records.

		bashKuberizeScript_Name := "bashKuberizeScript.sh"
		bashKuberizeScript, err := os.Create(bashKuberizeScript_Name)
		os.Chmod(bashKuberizeScript_Name, 0755)
		if err != nil {
			log.Fatal("Cannot create", bashKuberizeScript_Name, err)
		}
		defer bashKuberizeScript.Close()
		//Creating a bash script that runs the docker commands to make the image.
		fmt.Fprintln(bashKuberizeScript, "#!/bin/bash")
		//fmt.Fprintln(bashKuberizeScript, `kubectl create namespace`, clusterNameSpace)
		fmt.Fprintln(bashKuberizeScript, `gcloud container clusters create`, clusterName, `--zone`, clusterZone)
		fmt.Fprintln(bashKuberizeScript, `docker tag`, dockerCustomImageNAME+":"+dockerCustomImageVERSION, `gcr.io/`+gcloudProject+`/`+dockerCustomImageNAME+":"+dockerCustomImageVERSION)
		fmt.Fprintln(bashKuberizeScript, `gcloud docker -- push gcr.io/`+gcloudProject+`/`+dockerCustomImageNAME+":"+dockerCustomImageVERSION)
		fmt.Fprintln(bashKuberizeScript, `kubectl run`, clusterName, `--image=gcr.io/`+gcloudProject+`/`+dockerCustomImageNAME+":"+dockerCustomImageVERSION)
		//fmt.Fprintln(bashKuberizeScript, `kubectl run`, clusterName, `--image=gcr.io/`+gcloudProject+`/`+dockerCustomImageNAME+":"+dockerCustomImageVERSION, `--namespace`, clusterNameSpace)
		fmt.Fprintln(bashKuberizeScript, `kubectl expose deployment`, clusterName, `--type LoadBalancer --port`, externalPort, `--target-port`, containerAppPort)
		fmt.Fprintln(bashKuberizeScript, `sleep 60`)
		fmt.Fprintln(bashKuberizeScript, `apt install snapd -y`)
		fmt.Fprintln(bashKuberizeScript, `helm init`)
		fmt.Fprintln(bashKuberizeScript, `kubectl describe services`, clusterName, `| grep 'LoadBalancer Ingress:' | awk '{print$3}'`)
		fmt.Fprintln(bashKuberizeScript, `clusterIP=$(kubectl describe services`, clusterName, `| grep 'LoadBalancer Ingress:' | awk '{print$3}')`)
		fmt.Fprintln(bashKuberizeScript, `gcloud dns --project=`+gcloudProject, `managed-zones create`, gcloudDnsZoneName, `--description= --dns-name=`+gcloudDomainName)
		fmt.Fprintln(bashKuberizeScript, `gcloud dns --project=`+gcloudProject, `record-sets transaction start --zone=`+gcloudDnsZoneName)
		fmt.Fprintln(bashKuberizeScript, `gcloud dns --project=`+gcloudProject, `record-sets transaction add $clusterIP --name=`+containerHostName+`.`+gcloudDomainName+`. --ttl=1500 --type=A --zone=`+gcloudDnsZoneName)
		fmt.Fprintln(bashKuberizeScript, `gcloud dns --project=`+gcloudProject, `record-sets transaction execute --zone=`+gcloudDnsZoneName)
		fmt.Fprintln(bashKuberizeScript, `sleep 60`)
		fmt.Fprintln(bashKuberizeScript, `podName=$(kubectl get pods | grep '`+serverNameNOSpacesLower+`' | ''awk '{print$1}')`)
		fmt.Fprintln(bashKuberizeScript, `helm install --name cert-manager --namespace kube-system stable/cert-manager`)
		fmt.Fprintln(bashKuberizeScript, `helm version`)
		fmt.Fprintln(bashKuberizeScript, `exit 0`)
		bashKuberizeScript.Close()
		//Executing a bash script that runs the docker commands to make the image.
		BASHScriptEXE(dockerCustomImagePath_IMAGES + "/" + bashKuberizeScript_Name)
	}
	/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
	/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

}

func BASHScriptEXE(bashScript string) {
	//BASHScriptEXE executes shell scripts, WITHOUT arguments.
	//The Job WILL FAIL IF YOU ADD Them!
	//EX. /Users/prestongoode/Documents/testFrom.sh.
	//This Will Execute The Script At The End of The Path.
	//YOU MUST SPECIFY THE ENTIRE PATH TO THE SCRIPT, INCLUDING THE SCRIPT!!
	cmd := exec.Command(bashScript)
	cmd.Stdin = strings.NewReader("")
	var out bytes.Buffer
	cmd.Stdout = &out
	err := cmd.Run()
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf(out.String())
}

//JSON Struct For Unmarshal
type Characters struct {
	Characters []Character `json:"users"`
}

type Character struct {
	Count    int         `json:"count"`
	Next     string      `json:"next"`
	Previous interface{} `json:"previous"`
	Results  []struct {
		Name      string   `json:"name"`
		Height    string   `json:"height"`
		Mass      string   `json:"mass"`
		HairColor string   `json:"hair_color"`
		SkinColor string   `json:"skin_color"`
		EyeColor  string   `json:"eye_color"`
		BirthYear string   `json:"birth_year"`
		Gender    string   `json:"gender"`
		Homeworld string   `json:"homeworld"`
		Films     []string `json:"films"`
		Species   []string `json:"species"`
		Vehicles  []string `json:"vehicles"`
		Starships []string `json:"starships"`
		URL       string   `json:"url"`
	} `json:"results"`
}
