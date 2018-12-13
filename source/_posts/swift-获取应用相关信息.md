---
title: Swift 获取应用相关信息
date: 2018-12-10 13:33:10
tags: Swift
---
有时我们在 App 中提交一些统计信息或者用户反馈信息时，为了能更好地进行分析，通常会附带上当前应用程序的名称、版本号、设备型号、以及设备系统版本。
这一篇主要是记录一些自己遇到过或已知的方法，纯水文，没什么好讲的，直接展示代码：
```
//获取设备名称
let deviceName = UIDevice.current.name
print("deviceName:\(deviceName)")
//获取系统名称
let sysName = UIDevice.current.systemName
print("sysName:\(sysName)")
//获取系统版本
let sysVersion = UIDevice.current.systemVersion
print("sysVersion:\(sysVersion)")
//获取设备唯一标识符
let deviceUUID = UIDevice.current.identifierForVendor?.uuidString
print("deviceUUID:\(deviceUUID!)")
//获取设备的型号
let deviceModel = UIDevice.current.model
print("deviceModel:\(deviceModel)")
//电池电量
//UIDevice.current.isBatteryMonitoringEnabled 方法必须使用
UIDevice.current.isBatteryMonitoringEnabled = true
let batteryLevel = UIDevice.current.batteryLevel
print("batteryLevel:\(batteryLevel)")
//电池状态
let batteryState = UIDevice.current.batteryState
switch batteryState {
case .unknown: print("未识别")
case .charging: print("充电中")
case .full: print("充满状态")
case .unplugged: print("非充电状态")
}
UIDevice.current.isBatteryMonitoringEnabled = false


let infoDictionary = Bundle.main.infoDictionary!
//app版本号
if let appVersion = infoDictionary["CFBundleVersion"]{
print("appVersion:\(appVersion)")
}

//app名称
if let appName = infoDictionary["CFBundleDisplayName"]{
print("appName:\(appName)")
}

//主程序版本号
if let shortVersion = infoDictionary["CFBundleShortVersionString"]{
print("shortVersion:\(shortVersion)")
}
```
获取手机具体型号：
```
extension UIDevice{
    var deviceName: String{
        var systemInfo = utsname()
        uname(&systemInfo)

        let platform = withUnsafePointer(to: &systemInfo.machine.0) { ptr in
        return String(cString: ptr)
        }
        switch platform {
        case "iPhone3,1", "iPhone3,2", "iPhone3,3":  return "iPhone 4"
        case "iPhone4,1":  return "iPhone 4s"
        case "iPhone5,1":  return "iPhone 5"
        case "iPhone5,2":  return "iPhone 5 (GSM+CDMA)"
        case "iPhone5,3":  return "iPhone 5c (GSM)"
        case "iPhone5,4":  return "iPhone 5c (GSM+CDMA)"
        case "iPhone6,1":  return "iPhone 5s (GSM)"
        case "iPhone6,2":  return "iPhone 5s (GSM+CDMA)"
        case "iPhone7,2":  return "iPhone 6"
        case "iPhone7,1":  return "iPhone 6 Plus"
        case "iPhone8,1":  return "iPhone 6s"
        case "iPhone8,2":  return "iPhone 6s Plus"
        case "iPhone8,4":  return "iPhone SE"
        case "iPhone9,1":  return "国行、日版、港行iPhone 7"
        case "iPhone9,2":  return "港行、国行iPhone 7 Plus"
        case "iPhone9,3":  return "美版、台版iPhone 7"
        case "iPhone9,4":  return "美版、台版iPhone 7 Plus"
        case "iPhone10,1", "iPhone10,4":   return "iPhone 8"
        case "iPhone10,2", "iPhone10,5":   return "iPhone 8 Plus"
        case "iPhone10,3", "iPhone10,6":   return "iPhone X"
        case "iPhone11,2":   return "iPhone XS"
        case "iPhone11,4", "iPhone11,6":   return "iPhone XS MAX"
        case "iPhone11,8":   return "iPhone XR"


        case "iPad1,1":   return "iPad"
        case "iPad1,2":   return "iPad 3G"
        case "iPad2,1", "iPad2,2", "iPad2,3", "iPad2,4":   return "iPad 2"
        case "iPad2,5", "iPad2,6", "iPad2,7":  return "iPad Mini"
        case "iPad3,1", "iPad3,2", "iPad3,3":  return "iPad 3"
        case "iPad3,4", "iPad3,5", "iPad3,6":  return "iPad 4"
        case "iPad4,1", "iPad4,2", "iPad4,3":  return "iPad Air"
        case "iPad4,4", "iPad4,5", "iPad4,6":  return "iPad Mini 2"
        case "iPad4,7", "iPad4,8", "iPad4,9":  return "iPad Mini 3"
        case "iPad5,1", "iPad5,2":  return "iPad Mini 4"
        case "iPad5,3", "iPad5,4":  return "iPad Air 2"
        case "iPad6,3", "iPad6,4":  return "iPad Pro 9.7"
        case "iPad6,7", "iPad6,8":  return "iPad Pro 12.9"
        case "iPad6,11", "iPad6,12":  return "iPad 5"
        case "iPad7,1", "iPad7,2":   return "iPad Pro 12.9-inch 2nd-gen"
        case "iPad7,3", "iPad7,4":   return "iPad Pro 10.5"
        case "iPad7,5", "iPad7,6":   return "iPad 6"


        case "AppleTV2,1":  return "Apple TV 2"
        case "AppleTV3,1", "AppleTV3,2":  return "Apple TV 3"
        case "AppleTV5,3":   return "Apple TV 4"
        case "i386", "x86_64":   return "Simulator"

        case "iPod1,1":  return "iPod Touch 1"
        case "iPod2,1":  return "iPod Touch 2"
        case "iPod3,1":  return "iPod Touch 3"
        case "iPod4,1":  return "iPod Touch 4"
        case "iPod5,1":  return "iPod Touch (5 Gen)"
        case "iPod7,1":  return "iPod Touch 6"

        default:  return platform

        }

    }
}
```
直接使用
```
UIDevice.current.deviceName
```
还有一些方法没有记下，日后遇到再补上。

[demo地址](https://github.com/darren1192/DeviceInfoDemo)
