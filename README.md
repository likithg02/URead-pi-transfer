# URead-pi-transfer
ANDROID FILE TRANSFER APPLICATION
WORKFLOW DOCUMENT
________________________________________
1. APPLICATION WORKFLOW
1.1 Launch and Navigation
When the application starts, it opens with MainActivity, which serves as the central navigation hub and provides access to:
•	Send Files
•	Receive Files
•	Local File Manager
•	Setup / Configuration
•	Settings (WiFi / Bluetooth access)
The user selects the desired operation from this home interface.
________________________________________
1.2 File Sending Workflow
1.	The user opens the Send File module.
2.	The system offers two options:
•	Select a single file
•	Select an entire folder
3.	The Android Storage Access Framework (SAF) is invoked using:
•	ACTION_OPEN_DOCUMENT for single file selection
•	ACTION_OPEN_DOCUMENT_TREE for folder selection
4.	File metadata is extracted, including:
•	Filename
•	Size
•	MIME type
5.	The application retrieves the selected file’s InputStream via ContentResolver.
6.	SessionManager provides server configuration:
•	Host
•	Port
•	Username
•	Password
7.	A secure SFTP connection is created (using JSch).
8.	The file is uploaded to the remote server using a JSch SFTP channel.
9.	Background threads handle the upload; the main thread updates:
•	Progress bar
•	Status messages
10.	On successful transfer:
•	A system notification confirms completion.
________________________________________
1.3 File Receiving Workflow
1.	The user opens the Receive module.
2.	The app connects to the server using stored credentials from SessionManager.
3.	A list of available files in the remote directory is fetched.
4.	These files are displayed using a RecyclerView.
5.	When the user selects a file:
•	It is downloaded to a local storage directory.
6.	A notification confirms successful download.
7.	The downloaded file becomes visible inside the Local File Manager.
________________________________________
1.4 Local File Manager Workflow
1.	The user opens the Local File Manager (FilesActivity).
2.	Local files and directories are displayed using a custom data model (e.g., FSItem).
3.	On selecting an item:
•	If it is a directory: its contents are displayed.
•	If it is a file: it is opened in PreviewActivity.
4.	PreviewActivity allows the user to:
•	View file information
•	Download or export to another location
•	Delete the file
5.	All operations immediately update the local filesystem and the displayed list.
________________________________________
1.5 Setup Workflow
1.	The user opens SetupActivity.
2.	The user enters server details:
•	Host
•	Port
•	Username
•	Password
3.	SessionManager stores these details securely using SharedPreferences.
4.	All send / receive modules automatically read credentials from SessionManager.
________________________________________
1.6 Wireless Configuration Workflow
1.	If WiFi is disabled and the user tries to send or receive files, the app shows a prompt.
2.	When the user selects Configure WiFi, the app opens:
•	Settings.ACTION_WIFI_SETTINGS
3.	Bluetooth settings are handled similarly when Bluetooth-related functionality is used.
________________________________________
2. ISSUES ENCOUNTERED AND SOLUTIONS IMPLEMENTED
2.1 File Path and Metadata Issues
•	Problem: SAF returned URIs without real filesystem paths.
•	Cause: Android 10+ storage restrictions.
•	Solution: Use ContentResolver to obtain file metadata and InputStream instead of relying on raw filesystem paths.
________________________________________
2.2 SFTP Upload Stuck at 0%
•	Problem: Upload progress remained at 0%.
•	Cause: UI updates were attempted from a background thread.
•	Solution: Use Handler(Looper.getMainLooper()) (or similar main-thread mechanisms) to safely update UI elements.
________________________________________
2.3 Notifications Not Displaying on Android 13+
•	Problem: Transfer success notifications did not appear.
•	Cause: Missing NotificationChannel.
•	Solution: Implement a NotificationChannel with:
•	Proper channel ID
•	Name
•	Importance level
and register it before posting notifications.
________________________________________
2.4 “lateinit property not initialized” Crash
•	Problem:
kotlin.UninitializedPropertyAccessException: lateinit property rootScroll has not been initialized
•	Cause: UI element accessed before findViewById (or view binding) initialization.
•	Solution: Initialize rootScroll inside onCreate (after setContentView) before usage.
________________________________________
2.5 Folder Selection Not Working
•	Problem: Could not pick folders for upload.
•	Cause: Incorrect intent for folder selection.
•	Solution: Use:
•	ACTION_OPEN_DOCUMENT_TREE
for directory selection via SAF.
________________________________________
2.6 Server Credentials Not Saving
•	Problem: Setup values kept resetting.
•	Cause: Incorrect SharedPreferences mode and missing .apply() or .commit().
•	Solution:
•	Use MODE_PRIVATE for SharedPreferences.
•	Call .apply() (or .commit()) after editing.
________________________________________
2.7 Incorrect Remote Directory Handling
•	Problem: Uploaded files appeared in the wrong server folder.
•	Cause: Unsafe or incorrect string concatenation for SFTP paths.
•	Solution: Implement safe directory joining logic that:
•	Avoids duplicate slashes
•	Ensures correct base directory usage
________________________________________
2.8 WiFi Settings Not Opening
•	Problem: “Configure WiFi” opened the wrong settings screen.
•	Cause: Incorrect intent action used.
•	Solution: Use:
•	Intent(Settings.ACTION_WIFI_SETTINGS)
________________________________________
2.9 Downloaded Files Not Visible in File Manager
•	Problem: Files were downloaded but not visible to the user.
•	Cause: MediaStore was not updated.
•	Solution:
•	Write the file to a correct external storage path.
•	Notify MediaStore of the new file so it shows up in file managers and galleries.
________________________________________
3. EXPLANATION OF EACH LAYOUT
3.1 activity_main.xml
•	Purpose: Central navigation hub of the app.
•	Contains:
•	Buttons: Send, Receive, Local Files, Setup
•	Simple, clean layout for module selection
•	Layout Type: LinearLayout or ConstraintLayout.
________________________________________
3.2 activity_send.xml
•	Features:
•	Button to choose file
•	Button to choose folder
•	TextView showing selected file / folder name
•	ProgressBar for upload progress
•	Upload button
•	Status indicator at the bottom (e.g., “Uploading…”, “Completed”)
________________________________________
3.3 activity_receive.xml
•	Contains:
•	RecyclerView listing remote files from the server
•	Refresh button to reload the file list
•	Download button inside each item layout
•	Progress indicator (e.g., ProgressBar)
________________________________________
3.4 item_file.xml (for RecyclerView items)
•	Used for each remote file entry.
•	Contains:
•	File icon
•	File name
•	File size
•	Download button
________________________________________
3.5 activity_files.xml (Local File Manager)
•	Layout includes:
•	ListView or RecyclerView showing local FSItem objects
•	Path indicator (current directory)
•	Back navigation icon or button
________________________________________
3.6 activity_preview.xml
•	Components:
•	TextView for file name and metadata
•	ImageView or placeholder for previews (images, icons, etc.)
•	Button to download/export
•	Button to delete file
•	ScrollView for long text content
________________________________________
3.7 activity_setup.xml
•	Contains:
•	EditText fields for:
•	Host
•	Port
•	Username
•	Password
•	Switch or checkbox for “Remember settings”
•	Save button to store configuration via SessionManager
________________________________________
3.8 activity_wifi_settings.xml (Optional / Placeholder)
In many projects, there is no separate layout; instead, a button triggers a system settings intent.
•	If present, may contain:
•	Button or card to open WiFi settings using Settings.ACTION_WIFI_SETTINGS
•	Possibly a note or helper text explaining why WiFi is required

















