# YHiOSPackage

### how to use:

0. multiple configuration:[http://www.jianshu.com/p/83b6e781eb51,https://github.com/appfoundry/ios-multi-env-configuration](http://www.jianshu.com/p/83b6e781eb51,https://github.com/appfoundry/ios-multi-env-configuration) , or you just can use Debug || Release
1. install xcode-select:xcode-select --install
2. remove & change gem source(if your gem source is not this)(目前国内的gem source好像安装不了fastlane2.0,可安装完后再改回来): remove: gem source -r XXX   add: gem source -a [https://rubygems.org/](https://rubygems.org/)
3. install fastlane2.0:sudo gem install fastlane
4. push the .sh in the path of project
5. config the key:user, projectName, configuration, method, pgyerUKey, pgyerApiKey
6. execute .sh

```shell
#!/bin/bash

#how to use:
#0.multiple configuration:http://www.jianshu.com/p/83b6e781eb51,https://github.com/appfoundry/ios-multi-env-configuration , or you just can use Debug || Release
#1.install xcode-select:xcode-select --install
#2.remove & change gem source(if your gem source is not this)(目前国内的gem source好像安装不了fastlane2.0,可安装完后再改回来): remove: gem source -r XXX   add: gem source -a https://rubygems.org/
#3.install fastlane2.0:sudo gem install fastlane
#4.push the .sh in the path of project 
#5.config the key:user, projectName, configuration, method, pgyerUKey, pgyerApiKey
#6.execute .sh




#----------0.config
user=""                                           #your mac's userName                      
projectName=""									  #your project name

read -n 1 -p "[archive SIT(0) OR UAT(1)? input the number 0 || 1] : " mode

if [ $mode = "1" ] ;  then

configuration="UAT"
method='enterprise'                                 #xcodebuild method: app-store, package, ad-hoc, enterprise, development, developer-id

pgyerUKey=""        #pgyer's uKey
pgyerApiKey=""      #pgyer's _api_key

else

configuration="SIT"
method='development'

pgyerUKey=""
pgyerApiKey=""

fi

outputPath="/Users/${user}/Desktop/${configuration}-ipa"
mkdir ${outputPath}

#----------1.default config
#timer
SECONDS=0
now=$(date +"%Y%m%d%H%M%S")
projectPath=$(pwd)                                   #push the iOSPackage.sh in the path of project
scheme="${projectName}-${configuration}"
workspacePath="$projectPath/${projectName}.xcworkspace"
archivePath="$outputPath/${projectName}-${now}.xcarchive"
ipaName="${projectName}-${configuration}-${now}.ipa"
ipaPath="$outputPath/$ipaName"
appFullName="${projectName}-${configuration}"


echo "\n[archiving ${configuration}...]"

#----------2.pod update
#pod update --verbose --no-repo-update

#----------3.acchive&export
fastlane gym --workspace ${workspacePath} --scheme ${scheme} --clean --configuration ${configuration} --archive_path ${archivePath} --export_method ${method} --output_directory ${outputPath} --output_name ${ipaName}





#----------4.upload to pgyer
if [ -f $ipaPath ];
then
    echo "\n[Generate $ipaPath successfully!]"
    
    rm -rf ${archivePath}

    echo "[upload to pgyer]"
	curl -F "file=@${ipaPath}" -F "uKey=${pgyerUKey}" -F "_api_key=${pgyerApiKey}" http://www.pgyer.com/apiv1/app/upload --verbose

    echo "\n[Every boss, The ${appFullName}-${configuration} has been uploaded successfully!]"
else
    echo "\n[Generate $ipaPath fail!]"
    exit 1
fi





#----------5.end
echo "[Finished, total time: ${SECONDS}s]"
```

