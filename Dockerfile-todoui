FROM openjdk:8-alpine
RUN mkdir -p /opt/todoui
WORKDIR /opt/todoui
ENV JAVA_ARGS=""
COPY todoui/target/todoui-0.0.1-SNAPSHOT.jar /opt/todoui
#CMD ["java", "-jar", "todoui-0.0.1-SNAPSHOT.jar"]
CMD java $JAVA_ARGS -jar todoui-0.0.1-SNAPSHOT.jar