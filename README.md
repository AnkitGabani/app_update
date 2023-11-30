enum VersionError: Error {
    case invalidResponse, invalidBundleInfo
}

@discardableResult
func isUpdateAvailable(completion: @escaping (Bool?, Error?) -> Void) throws -> URLSessionDataTask {
    guard let info = Bundle.main.infoDictionary,
        let currentVersion = info["CFBundleShortVersionString"] as? String,
        let identifier = info["CFBundleIdentifier"] as? String,
        let url = URL(string: "http://itunes.apple.com/lookup?bundleId=\(identifier)") else {
            throw VersionError.invalidBundleInfo
    }
        
    let request = URLRequest(url: url, cachePolicy: URLRequest.CachePolicy.reloadIgnoringLocalCacheData)
    
    let task = URLSession.shared.dataTask(with: request) { (data, response, error) in
        do {
            if let error = error { throw error }
            
            guard let data = data else { throw VersionError.invalidResponse }
                        
            let json = try JSONSerialization.jsonObject(with: data, options: [.allowFragments]) as? [String: Any]
                        
            guard let result = (json?["results"] as? [Any])?.first as? [String: Any], let lastVersion = result["version"] as? String else {
                throw VersionError.invalidResponse
            }
            completion(lastVersion > currentVersion, nil)
        } catch {
            completion(nil, error)
        }
    }
    
    task.resume()
    return task
}

***********************************************************



    func applicationWillEnterForeground(_ application: UIApplication) {
        
        try? isUpdateAvailable {[self] (update, error) in
            if let error = error {
                print(error)
            } else if update ?? false {
                // show alert
                
                DispatchQueue.main.async {
                    let alertMessage = "An update to the app is required to continue.\nPlease go to the app store and upgrade your application."

                    let alertController = UIAlertController(title: "You have new updates", message: alertMessage, preferredStyle: .alert)
     
                    let updateButton = UIAlertAction(title: "Update App", style: .default) { (action:UIAlertAction) in
                        guard let url = URL(string: "https://itunes.apple.com/app/id6448429453") else {
                            return
                        }
                        if #available(iOS 10.0, *) {
                            UIApplication.shared.open(url, options: [:], completionHandler: nil)
                        } else {
                            UIApplication.shared.openURL(url)
                        }
                    }

                    alertController.addAction(updateButton)
                    self.window?.rootViewController?.present(alertController, animated: true, completion: nil)
                }
                
            }
        }
    }


