# YHiOSPackage

```shell
#!/bin/bash

#how to use:
#0.multiple configuration:
#  http://www.jianshu.com/p/83b6e781eb51
#  https://github.com/appfoundry/ios-multi-env-configuration,
#  Otherwise you can only pack Debug || Release configuration

#1.install xcode-select:xcode-select --install

#2.remove & change gem source(if your gem source is not this)
#  (目前国内的gem source好像安装不了fastlane2.0以上,可安装完后再改回来，需要fq):
#  remove: gem source -r XXX
#  add: gem source -a https://rubygems.org/

#3.install the latest version fastlane: sudo gem install fastlane

#4.push the .sh in the path of project

#5.config the key:configuration, method, pgyerUKey, pgyerApiKey
#  You just have to configure these.
#  你只需要配置这4个必须的参数，其他的全都帮你搞定了，这或许是GitHub上配置最少，最高复用性的iOS打包脚本了。

#6.permissions: chmod +x ./YHiOSPackage.sh

#7.execute: ./YHiOPackage.sh

#----------0.config

projectName=`find *.xcodeproj -maxdepth 0`
projectName=${projectName%.*}

read -n 1 -p "[archive SIT(0) OR UAT(1) OR Release(2)? input the number 0 | 1 | 2] : " mode

if [ ${mode} = "0" ] ;  then

configuration="SIT"
method='development'                                #xcodebuild method: app-store, package, ad-hoc, enterprise, development, developer-id

pgyerUKey=""        #pgyer's uKey
pgyerApiKey=""      #pgyer's _api_key

elif [ ${mode} = "1" ] ;  then

configuration="UAT"
method='enterprise'

pgyerUKey=""
pgyerApiKey=""

else

configuration="Release"
method='enterprise'

pgyerUKey=""
pgyerApiKey=""

#get CFBundleShortVersionString
projectContentPath="./${projectName}"
plistPath=`find ${projectContentPath} -name "Info.plist"`
appVersion=`/usr/libexec/PlistBuddy -c "Print :CFBundleShortVersionString" ${plistPath}`

fi

#----------1.default config
#timer
SECONDS=0
now=$(date +"%Y%m%d%H%M%S")
projectPath=$(pwd)                                   #push the iOSPackage.sh in the path of project
if [ ${mode} = "2" ] ;  then
outputPath="/Users/${USER}/Desktop/${projectName}-${configuration}-ipa/${appVersion}"
scheme="${projectName}"
ipaName="${projectName}.ipa"
else
outputPath="/Users/${USER}/Desktop/${projectName}-${configuration}-ipa"
scheme="${projectName}-${configuration}"
ipaName="${projectName}-${configuration}-${now}.ipa"
fi
ipaPath="${outputPath}/${ipaName}"
workspacePath="${projectPath}/${projectName}.xcworkspace"
archivePath="${outputPath}/${projectName}-${now}.xcarchive"

echo -e "\n"
echo "[archiving ${configuration}...]"


#----------2.pod update
#pod update --verbose --no-repo-update


#----------3.acchive&export
fastlane gym --workspace ${workspacePath} --scheme ${scheme} --clean --configuration ${configuration} --archive_path ${archivePath} --export_method ${method} --output_directory ${outputPath} --output_name ${ipaName}


#----------4.upload to pgyer
if [ -f ${ipaPath} ];
then
echo -e "\n"
echo "[Generate ${ipaPath} successfully!]"

rm -rf ${archivePath}

echo -e "\n"

if [ ${mode} = "2" ] ;  then
echo "[upload to SVN]"
svn add ${outputPath}
svn commit -m "commit" ${outputPath}

fi

echo "[upload to pgyer]"
curl -F "file=@${ipaPath}" -F "uKey=${pgyerUKey}" -F "_api_key=${pgyerApiKey}" http://www.pgyer.com/apiv1/app/upload --verbose


echo -e "\n"
echo "[Every boss, The ${projectName}-${configuration} has been uploaded successfully!]"
else
echo -e "\n"
echo "[Generate ${ipaPath} fail!]"
exit 1
fi


#----------5.end
echo -e "\n"
echo "[Finished, total time: ${SECONDS}s]"
```

