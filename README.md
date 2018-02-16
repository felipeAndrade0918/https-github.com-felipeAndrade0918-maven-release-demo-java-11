# maven-release-demo

You can make a release using the following command:

`mvn -Dresume=false release:prepare release:perform -Darguments="-Dmaven.deploy.skip=true"`

This will skip the phase where the artifact is deployed to the Nexus.
