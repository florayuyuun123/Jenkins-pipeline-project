## Sample Dockerfile

```
# Use the official Tomcat 9 base image
FROM tomcat:9-jdk11-openjdk

# Remove default web applications
RUN rm -rf /usr/local/tomcat/webapps/*

# Copy the built WAR file into the Tomcat webapps directory
COPY target/*.war /usr/local/tomcat/webapps/ROOT.war

# Expose port 8080 for external access
EXPOSE 8080

# Start Tomcat server
CMD ["catalina.sh", "run"]

```
