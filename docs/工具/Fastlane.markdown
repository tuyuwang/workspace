---
layout: default
title: Fastlane
nav_order: 7
parent: 工具
---

brew install Fastlane

[安装蒲公英插件](https://www.pgyer.com/doc/view/fastlane)
fastlane add_plugin pgyer

## Fastlane

结构：
- Appfile: 配置bundleId，账号信息
- Deliverfile
- Fastfile: 编写打包功能
- Pluginfile
- README.md：自动添加Fastfile中的打包功能使用说明

### Fastfile

~~~sh

# 上传Appstore
def upload_appstore(ipa_path, fileName)
  desc "上传appstore"
  filePath = "#{ipa_path}/#{fileName}.ipa"
  sh("xcrun altool --upload-app -f  #{filePath}   -t ios --apiKey 4RX7888PY8 --apiIssuer  69a6de88-b78d-47e3-e053-5b8c7c11a4d1 --verbose")
end

# 打包
def generate_ipa(typePrefix,exportMethod,schemeName,configuration,output_name,output_directory)
    #say 'generate ipa'
    gym(
      workspace:PROJECT_FILE_PATH,
      scheme:"#{schemeName}",
      clean:'true',
      output_directory: "#{output_directory}",
      output_name: "#{output_name}.ipa",
      configuration: "#{configuration}",
      include_symbols:'true',
      include_bitcode:'false',
      archive_path:'./fastlane/fastlane_build/',
      export_method:"#{exportMethod}"
    )
end

def do_upload_dsym_bugly(app_key,app_id,ipa_name,ipa_path)
  fileName =  "#{ipa_name}.app.dSYM.zip"
  filePath = "#{ipa_path}/#{fileName}"

  filePath = filePath.gsub("~","")
  filePath = sh("echo /Users/${USER}#{filePath}")
  filePath = filePath.chomp

  #fileName = "vava-dash_ptest_v02.00.07.01_201807262125.app.dSYM.zip"
  #filePath = "/Users/romy/Documents/ipa/VAVA_Home/v1.0.0/dev/vava-home_dev_v01.00.00.12_201807271028.app.dSYM.zip"
  
  print "上传符号表...\n"

  sh("curl -k \"https://api.bugly.qq.com/openapi/file/upload/symbol?app_key=#{app_key}&app_id=#{app_id}\" \
--form \"api_version=1\" \
--form \"app_id=#{app_id}\" \
--form \"app_key=#{app_key}\" \
--form \"symbolType=2\" \
--form \"bundleId=#{APP_BUNDLE_ID}\" \
--form \"productVersion=#{APP_VER}\" \
--form \"channel=AppStore\" \
--form \"fileName=#{fileName}\" \
--form \"file=@#{filePath}\" --verbose --progress-bar")

  print "上传结束\n"
end


platform :ios do
  before_all do
    
   	#创建 ipa 输出 目录
	  sh("mkdir -p #{IAP_PATH_TEST}")
	  sh("mkdir -p #{IAP_PATH_DEMO}")
	  sh("mkdir -p #{IAP_PATH_PRO}")
	  sh("mkdir -p #{IAP_PATH_AppStore}")
  end

    #----------------------------------------------- 内测 -----------------------------------------------
	  desc "打 内测 包"
	  lane :NC do |options|

	  # cocoapods

	  do_set_build_number

	  output_name = get_output_name(DISPLAY_NAME,options,"test")
	   
	  generate_ipa('TestFlight',"ad-hoc",SCHEME_NAME_TEST,CONFIGURETION_NAME_Release,output_name,IAP_PATH_TEST)

	  sh("open #{IAP_PATH_TEST}")
	  
	  do_upload_dsym_bugly(BuglyAppKeyDebug,BuglyAppIDDebug,output_name,IAP_PATH_TEST)

	  end

	  #----------------------------------------------- 蒲公英 -----------------------------------------------
	  #未上传Dsym文件
	  desc "打 蒲公英 Debug 包"
	  lane :pgy_debug do
	  # 当终端输出需要带参数-allowProvisioningUpdates时，可以使用注释命令，平时建议不用
	  # build_app(export_method: "ad-hoc", export_xcargs: "-allowProvisioningUpdates")
	  build_app(export_method: "ad-hoc")

	  pgyer(api_key: "", user_key: "", password: "", install_type: "2")
	  end
	  
	  #未上传Dsym文件,主要用于flutter测试
	  desc "打 蒲公英 Release 包"
	  lane :pgy_release do
	  # 当终端输出需要带参数-allowProvisioningUpdates时，可以使用注释命令，平时建议不用
	  # build_app(export_method: "ad-hoc", export_xcargs: "-allowProvisioningUpdates")
	  build_app(export_method: "ad-hoc",  configuration: "Release",)

	  pgyer(api_key: "", user_key: "", password: "", install_type: "2")
	  end

	   #----------------------------------------------- App Store -----------------------------------------------
	  desc "打 App Store 包"
	  lane :deploy do |options|

	  do_set_build_number

	  output_name = get_output_name(DISPLAY_NAME,options,"AppStore")

	  generate_ipa('app-store',"app-store",SCHEME_NAME_AppStore,CONFIGURETION_NAME_Release,output_name,IAP_PATH_AppStore)

	  sh("open #{IAP_PATH_AppStore}")
	  upload_appstore(IAP_PATH_AppStore, output_name)
	  do_upload_dsym_bugly(BuglyAppKeyRelease,BuglyAppIDRelease,output_name,IAP_PATH_AppStore)
end

~~~