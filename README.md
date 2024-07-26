# aLex - augmented Legal experience
## Contexto
Las sentencias del Tribunal Constitucional están disponibles en su [portal de búsqueda](https://jurisprudencia.sedetc.gob.pe/sistematizacion-jurisprudencial/busqueda)

Este portal agrupa información de las sentencias de acuerdo permitiendo una búsqueda basada en la siguiente estructura.
```json
{
   "_id":"37385",
   "_index":"sentencias",
   "_score":null,
   "_source":{
      "attachment":{
         "content":"TRIBUNAL CONSTITUCIONAL \n\n• \n\nExp. ... \n\n• \n\n\n\t\t2017-04-11T16:25:14",
         "content_length":7784,
         "content_type":"application/pdf",
         "language":"es"
      },
      "fecha_publicacion":"1996-10-18",
      "fundamentos":[
         "Que el literal f) inciso veinticuatro, del artículo segundo de la Constitución Política del Estado, establece que nadie puede ser detenido sino por mandato escrito y motivado del Juez, o por orden policial en situaciones de flagrante delito, debiendo ser puesto el detenido a disposición del Juzgado dentro de las veinticuatro horas, o en el término de la distancia, aún en el supuesto que la detención se debiera a mandato judicial o flagrante delito.",
         "Que en el caso de autos don Mario Tito Uribe Zorrillo y doña Patricia Rubio Portocarrero, fueron detenidos contraviniéndose el precepto constitucional anteriormente mencionado; el hecho de haberse ordenado su libertad en mérito al recurso de habeas corpus, por la Jueza del Trigésimo Cuarto Juzgado Especializado en lo Penal de Lima, no altera la situación jurídica de los accionantes que fueron detenidos sin respetarse lo esblecido en la Ley veintitrés mil quinientos seis, Ley de Habeas Corpus y Amparo."
      ],
      "id":37385,
      "nombre_demandado":"Teniente de la Policía Nacional del Perú (Wilson Gálvez Arrascue) y otro",
      "nombre_demandante":"Martín Felipe Castro Talavera a favor de Mario Tito Zorrilla Uribe y otra",
      "numero_expediente":"00006-1996-HC",
      "sentencia_distrito":{
         "id":37,
         "nombre":"Distrito Judicial de Lima",
         "slug":"distrito-judicial-de-lima"
      },
      "sentencia_sala":{
         "id":3,
         "nombre":"Pleno",
         "slug":"pleno"
      },
      "slug":"00006-1996-hc",
      "url_archivo":"https://tc.gob.pe/jurisprudencia/1996/00006-1996-HC.pdf"
   },
   "_type":"_doc",
   "sort":[
      845596800000
   ]
}
```

Este tipo de estructura funciona bien para búsquedas básicas. Sin embargo, puede ser potenciada por el uso de LLM para extraer entidades nombradas y categorías adicionales.

## Procesamiento
El problema es que  `["_source"]["attachment"]["content"]` no contiene todo el contenido del documento sino únicamente una porción inicial. Además, no existe otro formato como HTML para extraer esta información. Por ello, es necesario utilizar algún modelo de visión computacional para extraer el texto de cada página de los archivos PDF que son imágenes escaneadas de los documentos originales. 

![Documento original disponible en PDF](resolucion.png "Documento original disponible en PDF")

En ese sentido se utilizó [DocTR](https://github.com/mindee/doctr) para extraer el texto y la posición de cada uno de las palabras en los documentos. Como resultado se tiene
![Documento original procesado por DocTR](3e9d3e69f3820cb473261c46c875645f4dbe01b52ff346f3bd6c94d0.png)

Cuya salida tiene la siguiente estructura:
```
document
└───page
│   │   dimensions
│   │   language
│   │   orientation
│   │   page_idx
│   │
│   └───block
│   │   │   geometry
│   │   └───line
│   │        │   geometry
│   │        └───word
│   │            │   geometry    
│   │            │   conficence    
│   │            │   value                    
│   │            ....
│   └───block
│       │   geometry
│       └───line
│           │   geometry
│           └───word
│               │   geometry    
│               │   conficence    
│               │   value                    
│               
```

## Reconstrucción del contenido
1. Luego del procesamiento para identificar palabras en el documento se ordenaron las líneas con el objetivo de reconstruir el orden y contenido inicial.
2. Se identificaron las secciones `ASUNTO`, `ANTECEDENTES`, `FUNDAMENTOS` y `FALLO` relevantes de cada sentencia.
3. Finalmente utilizamos esta nueva estructura por cada documento de sentencia.
```json
{
    "CONTEXTO": " 00006-1996-HC.pdf =================  [Documento original](https://tc.gob.pe/jurisprudencia/1996/00006-1996-HC.pdf) ---  TRIBUNAL CONSTITUCIONAL Exp. 006-96-HC/TC Caso: Martin Felipe Castro Talavera  SENTENCIA DEL TRIBUNAL CONSTITUCIONAL En Lima, a los siete dias del mes de agosto de mil novecientos noventa y seis, reunido en sesion de Pleno Jurisdiccional, el Tribunal Constitucional, con asistencia de los senores Magistrados: Nugent,  Presidente, Acosta Sanchez,  Vicepresidente, Aguirre Roca, Diaz Valverde, Rey Terry, Revoredo Marsano, Garcia Marcelo,  actuando como secretaria la doctora Maria Luz Vasquez, pronuncia la siguiente sentencia, vista en Audiencia Publica el dia de la fecha, luego de haber deliberado. ",
    "ASUNTO:": "Mediante Recurso Extraordinario, interpuesto por ante la Segunda Sala Penal de la Corte Superior de Justicia de Lima, se eleva para conocimiento del Tribunal Constitucional, el expediente sobre accion de HABEAS CORPUS, seguido por don MARTIN FELIPE CASTRO TALAVERA en favor de don MARIO TITO ZORRILLA y dona PATRICIA RUBIO PORTOCARRERO, contra WILSON GALVEZ ARRASCUE, TENIENTE PNP de la Direccion de Robo de Vehiculos (DIROVE) y el Oficial responsable en lafecha. ",
    "ANTECEDENTES:": "Confecha 13 de agosto de 1995 don Martin Felipe Castro Talavera, interpone accion de Habeas Corpus a favor de don Mario Tito Zorrilla Uribe y dona Patricia Rubio Portocarrero, en contra del Teniente PNP Wilson Galvez Arrascue y del Oficial a cargo en la fecha, por cuanto senala el recurrente que Sus representados se encuentran detenidos desde el once (11) de agosto de 1995 y al momento de la interposicion de la accion de garantia no han sido puestos aun a disposicion de la autoridadjudicial competente.  El 13 de agosto de 1995, admitida a tramite la accion de Habeas Corpus por detencion arbitraria, la Titular del Trigésimo Cuarto Juzgado Especializado en lo Penal de Lima, se constituyo en la delegacion de la Direccion de Robo de Vehiculos (DIROVE), entrevistandose con el Teniente PNP Wilson Aurelio Gâlvez Carrasco, quien refirio que los detenidos fueron denunciados ante la citada delegacion por la comision de delito contra el patrimonio, en la modalidad de apropiacion ilicita por don Luis Miguel Seminario Eléspuru; que a mérito de la referida denuncia se procede a intervenirlos el once de agosto trasladandolos a la delegacion de miraflores, contando con la presencia del representante del Ministerio Publico en las diligencias que se practicaron y en el levantamiento del Acta de Registro Personal y Domiciliario; preguntado por la razon de la detencion sin habérseles encontrado en flagrante delito, ni contado con una orden judicial, afirmo que esta se debio a una investigacion preliminar llevada a cabo los dias ocho, nueve y diez, dentro de la cual se establece que eran personas dificiles de ubicar y se presumia su responsabilidad.  Con fecha catorce de agosto de mil novecientos noventa y cinco la Juez de la causa expide la correspondiente resolucion declarando fundada la accion de habeas corpus interpuesta por don Martin Felipe Castro Talavera, a favor de don Mario tito Zorrilla Uribe y dona Patricia Rubio Portocarrero, contra el teniente de la PNP Wilson Aurelio Galvez Arrascue e INFUNDADA en cuanto se refiere al oficial responsable 0 jefe inmediato superior del accionado, por considerar que se ha verificado la detencion arbitraria de los referidos ciudadanos, los que fueron intervenidos el viernes 11 de agosto de 1995 por el Teniente Galvez Arrascue, quien en el transcurso de una investigacion policial en contra de los agraviados, y por Sul propia decision, sin tener mandato judicial ni encontrarse los investigados en flagrante delito, procedio a su intervencion, la misma que se produjo en la via priblica cuando transitaban por ella.  Entendiendo don Mario Tito Zorrilla Uribe que el Teniente PNP Wilson Aurelio Galvez Arrascue, no es el unico responsable de la detencion arbitraria de la que fueron objeto él y su esposa y apela de la Resolucion expedida por el Trigésimo Cuarto Juzgado Especializado en lo Penal de Lima; asimismo obra a fojas 61 la apelacion interpuesta por el Procurador Publico del Ministerio del Interior al cargo de los Asuntos Judiciales de la Policia Nacional del Peri.  La Segunda Sala Penal de Lima revoca la recurrida y reformandola declara infundada la Accion de Habeas Corpus, por considerar que no se puede soslayar el derecho de los que sienten afectado su patrimonio a presentar las denuncias que considere pertinentes, siendo obligacion de la Policia Nacional practicar las diligencias necesarias para esclarecer los hechos denunciados, desprendiéndose de autos que a mérito de la denuncia interpuesta por don Luis Miguel Seminario Eléspuru por delito contra el patrimonio, el accionado procedio a practicar las investigaciones del caso, con intervencion del Ministerio Publico y en acatamiento de las normas y disposiciones policiales vigentes; que con relacion a que no se puso a los detenidos a disposicion de la Fiscalia de Turno dentro de las 24 horas, obra el Parte Policial, sin nimero, de fecha 12 de agosto de 1995, que da cuenta de la conduccion de los detenidos a la Trigésimo Cuarta Fiscalia Provincial de Turno, a las trece horas del citado dia, habiéndose entrevistado el Sub Oficial a cargo con el encargado de la Mesa de Partes, quien manifesto que se habia ordenado que ese dia se recepcionaba solo hasta las doce y treinta horas razon por la cual los implicados en el hecho ilicito retornaron a la DIROVE.  Interpuesto el recurso de nulidad por don Mario Tito Zorrilla Uribe se senala que el recurso planteado debe entenderse como Recurso Extraordinario y, en consecuencia se eleva lo actuado al Tribunal Constitucional. ",
    "FUNDAMENTOS:": "Que el literal f) inciso veinticuatro, del articulo segundo de la Constitucion Politica del Estado, establece que nadie puede ser detenido sino por mandato escrito y motivado del Juez, 0 por orden policial en situaciones de flagrante delito, debiendo ser puesto el detenido a disposicion del Juzgado dentro de las veinticuatro horas, 0 en el término de la distancia, aun en el supuesto que la detencion se debiera a mandato judicial o flagrante delito.  Que en el caso de autos don Mario Tito Uribe Zorrillo y dona Patricia Rubio Portocarrero, fueron detenidos contraviniéndose el precepto constitucional anteriormente mencionado; el hecho de haberse ordenado Su libertad en mérito al recurso de habeas corpus, por la Jueza del Trigésimo Cuarto Juzgado Especializado en lo Penal de Lima, no altera la situacion juridica de los accionantes que fueron detenidos sin respetarse lo esblecido en la Ley veintitrés mil quinientos seis, Ley de Habeas Corpus y Amparo.  Que existen razones atendibles en el expediente relacionadas al accionar de la Policia Nacional, en el caso de autos.  Por estosj fundamentos; el Tribunal Constitucional; ",
    "FALLA:": "Revocando la sentencia apelada, su fecha veintidos de setiembre de mil novecientos noventicinco y, reformandola, confirmaron la de primera instancia, su fecha catorce de agosto de mil novecientos noventa y cinco, obrante de fojas cuarenta y cuatro a fojas cuarenta y ocho, que declara fundada la Accion de Habeas Corpus interpuesta por los accionantes; no siendo de aplicacion, en este caso, lo dispuesto en el articulo décimo primero de la Ley veintitrés mil quinientos seis.  S.S. NUGENT, ACOSTA SANCHEZ, AGUIRRE ROCA, REY TERRY, DIAZ VALVERDE, REVOREDO MARSANO, GARCIA MARCELO.   Dra. Maria Luz Vasquez Secretaria Relatora",
    "fecha_publicacion": "1996-10-18",
    "id": 37385,
    "nombre_demandado": "Teniente de la Policía Nacional del Perú (Wilson Gálvez Arrascue) y otro",
    "nombre_demandante": "Martín Felipe Castro Talavera a favor de Mario Tito Zorrilla Uribe y otra",
    "numero_expediente": "00006-1996-HC"
}
```

## Enriquecimiento de información
El rol de LLM en esta sección es importante porque. Cada sección será procesada con el fin de extraer mayor información como palabras clave y la extracción de entidades nombradas (NER). 
