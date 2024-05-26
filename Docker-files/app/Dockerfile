FROM openjdk:11 AS IMAGE
RUN apt update && apt install maven -y
RUN git clone https://github.com/inshabano06/Vprofile.git
RUN cd Vprofile && mvn install


FROM tomcat:9-jre11
LABEL "PROJECT"="Vprofile"
LABEL "Aurthor"="Insha"
RUN rm -rf /usr/local/tomcat/webapps/*
COPY --from=IMAGE Vprofile/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]
