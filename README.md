# TFPG_UOC16 - 2016/2017
##Aquest és el repositori de codi del Treball de Fi de Postgrau de la Júlia Planas Pons.
##En aquest repositori hi trobareu el codi R que s'ha utilitzat per dur a terme aquest Treball de fi de Postgrau.

##Codi del preprocessat del dataset inicial amb l'eina Rstudio:
###obrim el dataset i l'emmagatzemem dins la variable dataset:
dataset<-read.csv("dataset_1.csv", header=TRUE)

###executem les ordres head i summary per obtenir una visió general de què hi ha als camps del dataset:
head(dataset)

summary(dataset)

###substituïm el valor nul de l'atribut Ad.size pel valor "native":
dataset$Ad.size<-sub("^$", "native", dataset$Ad.size)

library(magrittr)
####tornem a convetir l’atribut en una variable categòrica:
dataset$Ad.size %<>% factor

###esborrem les observacions nul·les o imcomplertes:
dataset<-dataset[!is.na(dataset$Impressions),]

###creem una columna per la hora del dia a partir de l'atribut time_id:
dataset$hora<- substring(dataset$time_id,first=9, last=10)

###creem una columna i li emmagatzemem la informació del dia de la setmana:
dataset$dia<-substring(dataset$time_id,first=7,last=8)

dataset$dia[dataset$dia=="01"]<-"Saturday"

dataset$dia[dataset$dia=="02"]<-"Sunday"

dataset$dia[dataset$dia=="03"]<-"Monday"

dataset$dia[dataset$dia=="04"]<-"Tuesday"

dataset$dia[dataset$dia=="05"]<-"Wednesday"

dataset$dia[dataset$dia=="06"]<-"Thursday"

dataset$dia[dataset$dia=="07"]<-"Friday"

dataset$dia %<>% factor

names(dataset$dia)<-"dia_setmana"

###creem una columna i li emmagatzemem la informació del dia en format numèric:
dataset$dia<-substring(dataset$time_id,first=7,last=8)

###esborrem les files amb 0 impressions:
nullimps<-which(dataset$Impressions==0)

dataset<-dataset[-nullimps,]

###transformem la variable numèrica Clicks en una variable categòria amb 0 si no hi ha clic i 1 si hi a clic:
dataset$click_categ<-ifelse(dataset$Clicks==0,c("0"),c("1"))

###guardem el dataset en un document csv anomenat dataset_preprocessat:
write.csv(dataset, "dataset_preprocessat.csv", row.names=FALSE)
