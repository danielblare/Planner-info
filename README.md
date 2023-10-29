
# Planner - Keep things planned

!!! Original repository is private due to security/privacy reasons !!!

https://planner-support.com

Stay organized and on top of your schedule with Planner - the ultimate planner app that keeps everything in sync and makes planning a breeze! Whether you need to manage appointments, important events, or simply jot down your to-dos, Planner offers a comprehensive solution to help you stay on track and never miss a beat

## Key features

- Seamless Calendar with Smart Reminders: Enjoy a user-friendly calendar interface that allows you to effortlessly navigate through days, weeks, and months. Add and customize reminders with titles, notes, scheduled times, and personalized tags. Visualize your tasks with color-coded circles that make identification a breeze.
- Personalized Tags and Appearance: Customize your experience to reflect your unique style. Create, modify, and remove tags with distinct colors to organize your reminders. Moreover, personalize the overall appearance of the app to match your preferences and create a planner that's uniquely yours.
- Syncing across different devices: Sign in or create an account using your Google, Apple, or email credentials, and unlock the power of real-time syncing. Seamlessly access your planner from multiple devices, and watch as changes made on one device instantly appear on others. Your reminders, tags, and appearance settings stay perfectly mirrored in real-time, keeping you up-to-date no matter where you are.
- Convinient widgets for easy access: Maximize productivity with our range of widgets in various sizes. Quickly view essential information and easily manage your tasks without opening the app. The larger widgets display more details, providing a comprehensive overview of your schedule at a glance.

## Code features
- Sign up/in methods like Apple, Google, Email, Anonymous
- Account managing(reset password, delete, change profile pic, change name, link provider etc) using FirebaseAuth and FirebaseFirestore as a backend + Real time syncing'
- Customizable tags using FirebaseFirestore + Realtime syncing
- Designed and implemented custom calendar
- Realtime syncing of reminders with custom made efficient Combine logic
- Local files/cache managing for the best user performance
- Deeplinking allows to share reminders between devices
- Various custom components/extensions/managers that can be reused
- Push Notifications scheduling including handling database changes
- UI Appearance customization though user's settings + Realtime syncing
- Widgets supporting 3 families
- Dependency injection
- Haptic Feedback Service

## Screenshots

<div>
  <img src="https://github.com/stuffeddanny/Planner-Info/blob/main/Screenshots/mock.png?raw=true" alt="App Screenshot" width="250" />
  <img src="https://github.com/stuffeddanny/Planner-Info/blob/main/Screenshots/tags.png?raw=true" alt="App Screenshot" width="250" />
  <img src="https://github.com/stuffeddanny/Planner-Info/blob/main/Screenshots/appearance.png?raw=true" alt="App Screenshot" width="250" />
</div>

### From Design to Real implementation

![App Screenshot](https://github.com/stuffeddanny/Planner-Info/blob/main/Screenshots/widget_design.png?raw=true)
## Code snippets

### Password strenth component
<img src="https://github.com/stuffeddanny/Planner-Info/blob/main/Screenshots/password_strength.gif?raw=true" alt="App Screenshot" width="450" />

```swift
struct PasswordStrengthView: View {

    /// Warning messages for password strength indicator
    enum WarningMessage {
        
        // Strength levels
        case weak
        case medium
        case strong
        case veryStrong
        
        // Warning messages
        case short
        case unacceptableSymbols
        
        // Info
        case info
        
        /// Text description of the message
        var description: String {
            switch self {
            case .weak:
                return "Weak"
            case .medium:
                return "Medium"
            case .strong:
                return "Strong"
            case .veryStrong:
                return "Very strong"
            case .short:
                return "Enter at least \(Constants.minimumPasswordLengthRequired) symbols"
            case .unacceptableSymbols:
                return "Password contains unacceptable symbols or spaces"
            case .info:
                return "Password should be at least \(Constants.minimumPasswordLengthRequired) characters long.\nUse special symbols: \(String.specialSymbols.map { String($0) }.joined(separator: " "))"
            }
        }
        
        /// Amount of segments to be closed
        var segments: Int {
            switch self {
            case .weak:
                return 3
            case .medium:
                return 2
            case .strong:
                return 1
            case .veryStrong:
                return 0
            default:
                return 4
            }
        }
        
        /// Array of strength levels to exclude warning messages for iterating in ForEach
        static var strengthLevels: [WarningMessage] {
            [.weak, .medium, .strong, .veryStrong]
        }
        
        /// Color for the warning message text
        var messageColor: Color {
            switch self {
            case .short, .unacceptableSymbols:
                return .red
            default:
                return .primary
            }
        }
        
        /// Color for the level of strength segment
        var levelColor: Color {
            switch self {
            case .weak:
                return .red
            case .medium:
                return .orange
            case .strong:
                return .yellow
            case .veryStrong:
                return .green
            default:
                return .gray
            }
        }
    }

    /// Message set outside of the view by some password strength check logic
    @Binding var message: WarningMessage
    
    var body: some View {
        VStack(alignment: .trailing) {
            Segments
                .overlay {
                    GeometryReader { geo in
                        HStack {
                            // Pushes rectangle to the right
                            Spacer(minLength: 0)
                            
                            // Covering rectangle
                            Rectangle()
                                .foregroundColor(.gray)
                                .frame(width: CGFloat(message.segments) / 4 * geo.size.width)
                        }
                    }
                    // Making mask for overlay
                    .mask(Segments)
                }
            
            // Warning message
            Text(message.description)
                .font(.caption)
                .foregroundColor(message.messageColor)
                .multilineTextAlignment(.trailing)
                .animation(nil, value: message)
        }
    }
    
    private var Segments: some View {
        HStack {
            ForEach(WarningMessage.strengthLevels, id: \.self) { level in
                RoundedRectangle(cornerRadius: 10)
                    .frame(height: 6)
                    .foregroundColor(level.levelColor)
            }
        }
    }
}
```
### Deeplinks

```swift
/// Generates a deep link URL for the specified host type.
///
/// - Parameter scheme: The host type.
/// - Returns: A URL representing the deep link.
static func generateDeeplink(for scheme: Host) -> URL? {
    var components = URLComponents()
    components.scheme = "plannerapp"
    components.host = scheme.value
    var queryItems: [URLQueryItem] = []
    
    switch scheme {
    case .widget(let date):
        queryItems.append(URLQueryItem(name: "date", value: date.stringFormat))
    case .reminder(let reminder):
        queryItems.append(contentsOf: reminder.queryItems)
    }
            
    components.queryItems = queryItems
    return components.url
}

/// Provides query items for URL encoding.
var queryItems: [URLQueryItem] {
    var queryItems: [URLQueryItem] = []
    
    queryItems.append(URLQueryItem(name: CodingKeys.id.rawValue, value: id))
    queryItems.append(URLQueryItem(name: CodingKeys.date.rawValue, value: date.stringFormat))
    queryItems.append(URLQueryItem(name: CodingKeys.dateCreated.rawValue, value: dateCreated.stringFormat))
    queryItems.append(URLQueryItem(name: CodingKeys.title.rawValue, value: title))
    queryItems.append(URLQueryItem(name: CodingKeys.note.rawValue, value: note))
    if let tagId {
        queryItems.append(URLQueryItem(name: CodingKeys.tagId.rawValue, value: tagId))
    }
    if let notification {
        queryItems.append(URLQueryItem(name: CodingKeys.notification.rawValue, value: notification.stringFormat))
    }
    
    return queryItems
}

.onOpenURL { url in
  guard url.scheme == "plannerapp" else { return }
  switch url.host() {
    case "widget":
      guard let parameters = try? url.queryParameters(),
      let stringDate = parameters["date"],
      let date = try? Date(stringDate, strategy: .iso8601) else { return }
      Task {
        await viewModel.open(date)
      }
      return
    case "reminder":
      guard let parameters = try? url.queryParameters(),
      let reminder = Reminder(parameters: parameters) else { return }
      Task {
        remindersManager.add(reminder, for: reminder.date)
        await viewModel.open(reminder.date)
        try? await Task.sleep(for: .seconds(0.3))
        highlightedReminderID = reminder.id
      }
      return
    default: return
  }
}
```
### Database fetching using generics

```swift
/// Get a model from the database.
  func get<T: Codable>(for userID: String) async throws -> T {
        let user = firestore.collection(Keys.USERS_KEY)
            .document(userID)
        switch T.self {
        case is AppearanceSettingsModel.Type:
            return try await user
                .collection(Keys.SETTINGS_KEY)
                .document(Keys.APPEARANCE_SETTINGS_KEY)
                .getDocument(as: AppearanceSettingsModel.self) as! T
        case is TagsHolder.Type:
            return try await user
                .collection(Keys.SETTINGS_KEY)
                .document(Keys.TAGS_KEY)
                .getDocument(as: TagsHolder.self) as! T
        case is AuthUserModel.Type:
            return try await user
                .getDocument(as: AuthUserModel.self) as! T
        case is [RemindersHolder].Type:
            return try await user
                .collection(Keys.REMINDERS_KEY)
                .getDocuments()
                .documents.map { try $0.data(as: RemindersHolder.self) } as! T
        default: throw FirestoreErrorCode(.unimplemented)
        }
    }
```
### Asynchronous code

```swift
@discardableResult
func signIn(with option: SignInOption) async throws -> AuthUserModel {
    let credential: AuthCredential? = try await {
        switch option {
        case .email(let credential): return credential
        case .apple: return try await getAppleCredential()
        case .google: return try await getGoogleCredential()
        case .anonymous: return nil
        }
    }()
    
    let authDataResult = try await {
        if let credential {
            return try await Auth.auth().signIn(with: credential)
        } else {
            return try await Auth.auth().signInAnonymously()
        }
    }()
    return AuthUserModel(from: authDataResult.user)
}
```
### Database changes listening with custom logic implemented

```swift
/// Adds tags listener to see any realtime updated of user's reminders in the database
    private func addListener(for userID: String) {
        remindersListener = firebaseManager.addListener(for: userID, listenTo: .reminders) { [weak self] snapshot, error in
            guard let self else { return }
            // Making sure user has sync mode on before doing any changes
            guard UserDefaults.standard.getSyncOption(.remindersAndTags) == true else { return }

            self.tasks.insert(Task { @MainActor in
                if error != nil {
                    self.syncError = .sync
                } else if let snapshot {
                    guard let holders = try? snapshot.documentChanges.map({ try $0.document.data(as: RemindersHolder.self) }),
                          let _ = try? holders.map({ try Date($0.id, strategy: .iso8601) }) else {
                    self.syncError = .decoding
                        return
                    }
                    
                    self.resetError()
                    var updatesMade = false
                    for holder in holders {
                        guard let dateModified = self.remindersHolder[holder.dateId]?.dateModified, Calendar.current.compare(dateModified, to: holder.dateModified, toGranularity: .second) != .orderedAscending else {
                            updatesMade = true
                            withAnimation {
                                self.remindersHolder[holder.dateId] = holder
                            }
                            continue
                        }
                    }
                    
                    if updatesMade {
                        let reminders = self.remindersHolder.flatMap({ $0.value.reminders })
                        let remindersToNotify = reminders.filter({ $0.notification ?? .distantPast >= .now })
                        await NotificationService.shared.updatePendings(with: remindersToNotify)
                    }
                    
                    try? self.saveToLocalFiles(self.remindersHolder)
                }
            })
        }
    }
```

## Lessons Learned

Throughout the development of my iOS app, I've gained valuable insights and learned several important lessons that have contributed to the success of the project. These lessons can help guide me in future app development endeavors:

- User-Centered Design is Crucial: Putting the user at the center of the design process is paramount. Conducting user research, gathering feedback, and iterating based on user needs have consistently led to better user experiences.
- Testing, Testing, Testing: Rigorous testing is essential. I've learned to perform thorough testing on various devices and iOS versions to catch and address issues early, ensuring a smoother user experience.
- Maintain Code Quality: Writing clean, maintainable code pays off in the long run. Adhering to coding standards, using version control, and documenting code are critical for collaboration and future updates.
- Stay Updated with Apple Technologies: Apple's ecosystem evolves rapidly. Staying up-to-date with the latest iOS features, Swift updates, and design guidelines ensures the app remains relevant and competitive.
- Prioritize Performance: App performance directly impacts user satisfaction. Optimizing code, reducing unnecessary overhead, and efficiently managing resources contribute to a snappier and more responsive app.
- Accessibility Matters: Accessibility isn't an afterthought but an integral part of app development. Designing for accessibility not only makes the app more inclusive but also enhances its overall usability.
- Security is Non-Negotiable: Protecting user data is paramount. I've learned to implement robust security practices, including encryption, secure authentication, and regular security audits.
- Community and Documentation: Leveraging resources like Apple's developer forums and documentation is invaluable. Learning from the experiences of others and seeking help when needed can save time and effort.
- Continuous Improvement: App development is an ongoing process. I've embraced the idea of continuous improvement, regularly updating the app with new features, bug fixes, and performance enhancements.
- Adaptability is Key: The mobile landscape is dynamic. I've learned to be adaptable and open to change, whether it's in response to user feedback, emerging technologies, or market shifts.
- App Store Optimization (ASO): ASO techniques, such as keyword optimization and compelling app descriptions, play a vital role in app visibility and downloads. I've invested time in understanding and implementing ASO strategies.
By reflecting on these lessons learned, I can continue to enhance my skills, create better iOS apps, and deliver exceptional experiences to my users. App development is an ever-evolving journey, and each project teaches me something new

## 🛠 Skills
Swift, SwiftUI, MVVM, Async environment, Actors, Multithreading, WidgetKit, Combine, UserNotifications, Git, UX/UI, Webpage design, Deeplinks
Figma, FirebaseAuth, FirebaseFirestore, FirebaseStorage, Crashlytics, URLRequests, GoogleSignIn, Backend changes listening/syncing in real time

