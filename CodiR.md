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

dataset$hora %<>% factor

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

dataset$dia %<>% factor

###esborrem les files amb 0 impressions:
nullimps<-which(dataset$Impressions==0)

dataset<-dataset[-nullimps,]

###transformem la variable numèrica Clicks en una variable categòria amb 0 si no hi ha clic i 1 si hi a clic:
dataset$click_categ<-ifelse(dataset$Clicks==0,c("0"),c("1"))

dataset$click_categ %<>% factor


##transformem les variables time_id, Agency ID, Creative ID, Inventory ID, que R studio reconeix com a numèriques, en variables categòriques:

dataset$time_id %<>% factor

dataset$Agency.ID %<>% factor

dataset$Creative.ID %<>% factor

dataset$Inventory.ID %<>% factor

###guardem el dataset en un document csv anomenat dataset_preprocessat:
write.csv(dataset, "dataset_preprocessat.csv", row.names=FALSE)

##Codi de l'anàlisi mitjançant regressió logística del dataset:
###es crea un subset del dataset incloent només les variables que s'inclouran al model:
ds_regression<- subset(dataset,select=c(4,6,7,8,10,12,13,15,18,20,21))

###sobre el nou dataset es divideixen les dades en un set de test i un set d'entrenament:
require(caTools)

sample<-sample.split(ds_regression, SplitRatio = 0.7)

train<-subset(ds_regression, sample)

test<-subset(ds_regression, !sample)

###es crea el model amb la funció glm()

model <- glm(click_categ ~.,family=binomial(link='logit'),data=train)

##o be:

model_1<-train(click_categ ~ .,  data=train, method="glm", family="binomial")

###amb la comanda summary() es veuran els resultats de l'aplicació del model al set d'entrenament

##Avaluació de la bondat del model de regressió:
###es fan les prediccions
pred_1 <- predict(model_1, newdata=test)

#matriu de confusió
accuracy_1 <- table(pred_1, test[,"click_categ"])
confusionMatrix(data=pred_1,test$click_categ)

#accuracy
sum(diag(accuracy_1))/sum(accuracy_1)



##Codi per executar l'algoritme d'arbre de decisió sobre el dataset
###Dividim el fitxer en 70% entrenament i 30% validació
set.seed(1234)

ind <- sample(2, nrow(ds_regression), replace=TRUE, prob=c(0.7, 0.3))

trainData <- ds_regression[ind==1,]

testData <- ds_regression[ind==2,]

###Apliquem l'algoritme
ArbolRpart_ctree <- rpart(click_param~., method="class", data=trainData)

###Obtenima la relació de regles d'associació de l'arbre en format llista
print(ArbolRpart_ctree) # estadístiques detallades de cada node

###Obtenim l'arbre amb un disseny gràfic cuidat
f13<-rpart.plot(ArbolRpart_ctree,extra=4) #visualitzem l'arbre

summary(ArbolRpart_ctree) # estadístiques detallades de cada node

printcp(ArbolRpart_ctree) # estadístiques de resultats

plotcp(ArbolRpart_ctree) # evolució de l'error a mesura que s'incrementen els nodes

###Validem la capacitat de predicció de l'arbre amb el ftxer de validació
testPredRpart <- predict(ArbolRpart_ctree, newdata = testData, type = "class")

###Visualitzem la matriu de confusió
table(testPredRpart, testData$click_categ)

###Calculem el % d'encerts (Accuracy)
sum(testPredRpart == testData$click_param)/ length(testData$click_param)*100

