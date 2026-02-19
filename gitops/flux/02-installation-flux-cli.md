# Installation flux 

  * Das Kommandozeilen-Tool ist neben dem Operator der empfohlene Weg

## TRAINER: Schritt 1: Binary ausrollen (Hat Trainer bereits gemacht) 

```
sudo su -
```

```
curl -s https://fluxcd.io/install.sh | sudo bash
```

```
# Autocompletion aktivieren
flux completion bash > /etc/bash_completion.d/flux 
```


## Schritt 2: flux check --precheck (Voraussetzungen erfüllt ?)  

```
# Stimmen die Voraussetzungen ?
flux check --pre
```


## Schritt 3: gitlab - repo einrichten 

```
1. https://gitlab.com/projects/new#blank_project

```

<img width="1275" height="795" alt="image" src="https://github.com/user-attachments/assets/41e46904-3972-4c77-9be4-0f8960b268f3" />


```
# Klicken auf "Create Project"
```

> [!IMPORTANT] 
> Token am Ende rauskopieren 


## Schritt 4: Token für project erstellen 

```
1. https://gitlab.com/jmetzger/flux-test-jm/-/settings/access_tokens
```


```
* Api rechte vergeben.
* Maintainer Rolle wird mindestens benötigt, da flux, deploy keys verwaltet und repository Einstellungen setzen können muss.
```

<img width="1402" height="732" alt="image" src="https://github.com/user-attachments/assets/339149cc-0d4a-4f45-8f0a-bf26729445fa" />


## Schritt 5: Flux in gitlab bootstrappen  


```
# Endpunkt für die Flux - Konfigurationsobjekte einrichten
export GITLAB_TOKEN=<dein-personal-access-token>

# Bootstrap für Production-Cluster
flux bootstrap gitlab \
  --owner=training-tn<Deine-Teilnehmer-Nr> \
  --repository=<Dein-Repo> \
  --branch=main \
  --path=clusters/production \
  --token-auth
```

<img width="1290" height="783" alt="image" src="https://github.com/user-attachments/assets/6c1d1118-40c4-488d-9cfb-bd0b58a25e9c" />



