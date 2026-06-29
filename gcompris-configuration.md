# **Technical Reference and Configuration Architecture of the GCompris Educational Suite**

GCompris is a highly versatile, multi-activity educational software suite designed for children aged 2 to 10, developed under the umbrella of the KDE project and licensed under the GNU Affero General Public License (AGPLv3).1 Given its deployment across highly diverse educational environments—ranging from single-user home setups to massive, locked-down school computer laboratories—GCompris features a robust, multi-tiered configuration architecture.1  
System administrators and educators can manage and customize the software through three primary mechanisms: command-line parameters, persistent configuration files, and direct graphical user interfaces (GUI), including the specialized companion utility, gcompris-teachers.1 Understanding these configuration mechanisms is vital for optimizing graphical rendering, enforcing security via kiosk systems, and tailoring educational content to specific curricula.1

## **Command-Line and Console Parameter Configuration**

Invoking GCompris via the command-line terminal allows administrators to define runtime behavior, override persistent settings for a single session, and restrict user access without altering global configuration files.7 This is particularly critical in shared school environments where custom application launchers are distributed to thin clients or student workstations.7  
The modern Qt-based implementation (gcompris-qt) accepts several command-line arguments to handle display geometry, audio output, peripheral behavior, rendering engines, and localized execution.7

### **Modern Command-Line Interface Parameters**

The following table details the standard console parameters available in the contemporary Qt-based versions of GCompris:

| Long Option | Short Option | Parameter/Values | Functional Description |
| :---- | :---- | :---- | :---- |
| \--fullscreen | \-f | None | Launches GCompris in fullscreen mode, covering system panels.7 |
| \--window | \-w | None | Runs GCompris in a resizable window.7 |
| \--sound | \-s | None | Forces GCompris to run with sound enabled.7 |
| \--mute | \-m | None | Mutes all audio outputs during the session.7 |
| \--cursor | \-c | None | Enforces the use of the default operating system cursor.7 |
| \--nocursor | \-C | None | Disables the cursor entirely, optimizing the interface for touchscreen displays.7 |
| \--version | \-v | None | Outputs the current GCompris software version to the console.7 |
| \--list-activities | \-l | None | Prints a complete list of all available activities to standard output.7 |
| \--launch | None | \[activity\_name\] | Directly starts GCompris into the specified activity, bypassing the main menu.7 |
| \--start-level | None | \[level\_number\] | Specifies the starting level of the activity defined by \--launch.7 |
| \--difficulty | None | \[value\] or \[min-max\] | Restricts activity difficulty (values 1–6) for the current session.7 |
| \--enable-kioskmode | None | None | Activates kiosk mode, hiding both the Quit and Configuration buttons.7 |
| \--disable-kioskmode | None | None | Deactivates kiosk mode (default application behavior).7 |
| \--show-home-button | None | None | Displays the Home button on the navigation bar.8 |
| \--hide-home-button | None | None | Hides the Home button on the navigation bar.8 |
| \--renderer= | None | opengl, software, direct3d11, direct3d12, metal | Explicitly sets the graphics driver (Direct3D is Windows-only; Metal is macOS-only).8 |
| \--locale= | None | \[locale\_code\] | Runs GCompris under a specified language/locale code.8 |

Note: The older parameters \--software-renderer and \--opengl-renderer are deprecated and replaced by the unified \--renderer= switch.8

### **Legacy GTK-Based Command-Line Parameters**

In older deployments utilizing the legacy GTK+-based codebase of GCompris, command-line arguments differed significantly.9 These legacy options handled local directory overrides, database routing, profiling, and database creation 9:

| Legacy Parameter | Functional Equivalent / Purpose |
| :---- | :---- |
| \--config-dir | Directs the application to a custom configuration directory path, overriding the default $HOME/.config/gcompris or $XDG\_CONFIG\_HOME directory.9 |
| \--user-dir | Changes the default storage directory for student files, normally located at $HOME/My GCompris.9 |
| \--administration (-a) | Opens GCompris directly in administrative and user-management mode.9 |
| \--database (-b) | Specifies an alternate SQLite database path for profiles.9 |
| \--create-db | Creates the alternate SQLite profile database if it does not exist.9 |
| \--profile (-p) | Pre-selects a specific student profile configured via the administration menu.9 |
| \--profile-list | Lists all available profiles created using administrative tools.9 |
| \--disable-quit | Disables the quit button, preventing children from exiting GCompris.9 |
| \--disable-config | Hides the local configuration button on the navigation bar.9 |
| \--disable-level | Hides the level selection button on the navigation bar.9 |
| \--disable-database | Starts GCompris without database logging (slower start, no user log).9 |
| \--drag-mode | Sets the global drag-and-drop mechanism (normal, 2clicks, or both).9 |
| \--nolockcheck | Allows the execution of multiple parallel instances of GCompris.9 |
| \--no-zoom | Disables maximization zoom features.9 |
| \--timing-base | Increases default activity timeout delays (useful values are greater than ![][image1]).9 |
| \--timing-mult | Adjusts how activity timeout delays scale for several actors (useful values are less than ![][image1]).9 |
| \--test | Runs a continuous diagnostic loop across all activities.9 |
| \--debug (-D) | Displays comprehensive diagnostic information on the console standard output.9 |

Legacy versions allowed advanced temporal adjustment via timing-related command-line switches.9 For instance, if the default hardcoded timeout delay for an activity actor is ![][image2], the modified runtime delay ![][image3] is mathematically calculated based on the timing base parameter ![][image4] and the multiplier ![][image5], scaling with the number of active entities ![][image6]:  
![][image7]  
This adjustment allows administrators to adapt the suite's pace for children with specific cognitive or motor needs.9

## **Filesystem-Level Configuration Files and Storage Paths**

When configuration parameters are modified via the GUI, or when administrators need to scale settings across multiple identical computers, GCompris utilizes persistent, plain-text configuration files.1 These files are written using standard key-value pairings (often structured as INI files through the Qt configuration framework) and can be parsed or edited manually using any standard text editor.4

### **Configuration File Paths**

The storage directory and file name depend heavily on the target operating system, the underlying architecture (legacy GTK+ vs. modern Qt), and the packaging format 4:

| Operating System / Packaging | Configuration File Path |
| :---- | :---- |
| **GNU/Linux (Native/Distro Packages)** | \~/.config/gcompris-qt/gcompris-qt.conf or \~/.config/gcompris/gcompris-qt.conf 4 |
| **GNU/Linux (Flatpak)** | \~/.var/app/org.kde.gcompris/config/gcompris/gcompris-qt.conf 11 |
| **Microsoft Windows (Desktop)** | %LocalAppData%\\gcompris\\GCompris.conf 6 |
| **macOS** | \~/Library/Preferences/gcompris-qt.conf or \~/Library/Application Support/gcompris/gcompris-qt.conf 12 |

### **Practical Configuration Use-Cases: Graphics Rendering**

One of the most frequent manual interventions involving GCompris configuration files relates to graphical rendering drivers.6 GCompris requires 32-bit color depth and a graphics processing unit that supports OpenGL 2.1 or higher.1 On newer builds, GCompris defaults to OpenGL or Direct3D 11 rendering and attempts to fall back automatically to software rendering if standard drivers crash or fail to initialize.6  
However, in virtualized school environments or on legacy computer hardware, this auto-detection routine can fail or trigger system hangs.6 To bypass this, system administrators can force software rendering manually by opening the configuration file, locating the renderer line, and modifying it:

Ini, TOML  
\# Forcing software-based rendering in gcompris-qt.conf or GCompris.conf  
renderer\=software

Replacing renderer=auto with renderer=software bypasses hardware acceleration entirely, allowing the application to utilize the system's central processing unit for all graphical operations.6

### **Decentralized Asset Caching**

To keep core installers lightweight and conserve bandwidth during initial deployments, GCompris does not ship with full-resolution images, localized voices, or background audio files for all supported languages.16 Instead, the system downloads these resources on demand, saving them locally in an OS-specific cache directory.17  
For environments without internet access or with strict enterprise firewalls, administrators can manually populate these cache subfolders with pre-downloaded .rcc asset packages and matching Contents files.17 The precise path to the cache directory varies by platform:

* **GNU/Linux:** /home/userName/.cache/KDE/gcompris-qt/data3/ 17  
* **Microsoft Windows:** C:\\Users\\Username\\AppData\\Local\\KDE\\GCompris\\cache\\data3\\ 17  
* **macOS:** \~/Library/Caches/KDE/gcompris-qt/data3/ 17  
* **Android (Root access required):** /data/data/net.gcompris.full/data3/ 17

To add assets manually, localized voice and image packages are sorted into subdirectories named voices-codecName, backgroundMusic, and words inside the cache path.17 Standard audio codecs are selected based on the host operating system: .ogg for Linux and Android, .mp3 for Windows, and .aac for macOS.17  
Furthermore, versions prior to GCompris 4.0 utilized the legacy data2 path, whereas versions starting from 4.0 write to the modern data3 directory.17 Modern versions also append timestamp indicators directly to .rcc filenames, whereas older structures used static naming conventions 17:

| Resource Type | Pre-v4.0 Naming Convention | Post-v4.0 Naming Convention (Example) |
| :---- | :---- | :---- |
| **Background Music** | backgroundMusic-ogg.rcc 17 | backgroundMusic-ogg-2024-03-19-11-10-30.rcc 17 |
| **Voices (Localized)** | voices-localeCode.rcc 17 | voices-localeCode-2024-03-19-11-10-30.rcc 17 |
| **Words (Images)** | words-webp.rcc 17 | words-webp-2024-03-19-11-10-30.rcc 17 |

## **Graphical User Interface Configuration**

For typical home users and classroom instructors, configuration is managed directly through the user-friendly control bar located at the bottom of the GCompris main screen.1 This control bar contains two primary customization options: the global Configuration Menu and the localized Activity Settings Menu.1

### **The Toolbox: Global Configuration Menu**

Represented by a toolbox icon, this menu allows users to modify overall system preferences 1:

* **Language Selection:** Overrides the system default locale to run GCompris in a target language, supporting dual-language and foreign language classrooms.8  
* **Audio Controls:** Toggles background music and sound effects on or off.7  
* **Asset Management:** Allows manual triggering of external asset downloads (such as the "Full word image set") directly through the GUI.16  
* **Teacher Connection:** Provides a dedicated sub-panel at the bottom of the interface to connect the client instance to a local gcompris-teachers server.5

These settings are automatically saved to the active user's persistent configuration file upon exiting the menu.1

### **The List Item: Activity Settings Menu**

Represented by a three-line list icon, this menu provides fine-grained, pedagogical controls over the currently running activity.4 The interface is split into two tabs:

* **Dataset Tab:** Allows educators to filter activity content to align with specific learning objectives.1 For example, in numeracy exercises, teachers can restrict the mathematical range to smaller integers (e.g., numbers up to 10\) for early learners, leaving higher numbers for advanced groups.3  
* **Options Tab:** Enables structural changes within the current activity.1 Examples include substituting generic graphics with high-resolution photographic images (such as using real car photos in traffic activities) or altering the font and keyboard layouts to match localized standards.5

## **Enterprise and Classroom Management: GCompris Teachers**

In structured classroom environments, configuring hundreds of individual GCompris clients manually is highly inefficient. To address this, the software suite includes gcompris-teachers.5 This companion utility acts as a central administration panel, allowing teachers to manage student profiles, create customized datasets, and track real-time activity results.5

### **Database and User Profiles**

When launching gcompris-teachers, the educator is prompted to establish a teacher login and associated database.5 To prevent unauthorized access to sensitive student performance data, the application supports database-level encryption secured by the teacher's master password.5 Within this dashboard, the database can be populated through several built-in management tools:

* **Roster Generation:** Educators can add individual pupils, assign unique login passwords, and organize them into specific groups or classes.5  
* **CSV Bulk Import:** To streamline initial setups, rosters can be imported in bulk using structured Comma-Separated Value (CSV) files.5  
* **Dataset Management:** Teachers can customize activity configurations and package them as specific learning objectives or "work plans".5 For example, customized spelling lists or arithmetic difficulty caps can be assigned to specific target groups.3

### **Server-Client Network Configuration**

To distribute configured lesson plans and aggregate student performance data, gcompris-teachers establishes a lightweight server-client architecture over the local area network (LAN).5 This requires coordinated configuration on both the server and client devices:

#### **Server Configuration**

On the teacher's administrative workstation, the educator navigates to the Settings page within gcompris-teachers and defines two parameters:

1. **Server ID:** A unique alphanumeric identifier used to distinguish the teacher's system on the local subnet.5  
2. **Network Port:** The TCP/IP port used to listen for client connections.5 If no port conflicts exist on the host system, the default port is recommended.5

#### **Client Configuration**

On each student device running GCompris, the administrator or instructor must access the global Configuration Menu, navigate to the bottom, and click the **Teacher's server config** button.5 The client's fields must be set to match the server precisely:

Ini, TOML  
\# Required connection parameters on client devices  
ServerID\=  
Port=

To ensure reliable communications, local firewalls on both the server and client systems must be configured to allow inbound and outbound traffic through the designated TCP port.5

#### **Connection Workflow**

Once configured, the daily synchronization process operates through a simple, coordinated workflow:

1. **Server Initialization:** The teacher launches gcompris-teachers, filters the active group, and clicks the **Connect devices** button.5  
2. **Client Connection:** Students start GCompris and remain on the main menu.5 When the teacher initiates the connection, client devices display a prompt to confirm the network connection.5  
3. **Authentication:** The teacher selects the appropriate usernames and sends the login list to the connected devices.5 Students select their names, enter their passwords, and authenticate.5  
4. **Real-Time Monitoring:** Successful authentication is shown on the teacher's console through color-coded status indicators (Green: connected, Red: authentication failure, Grey: disconnected, Transparent: idle).5 The server then pushes the custom dataset to the students, allowing them to complete the lesson while transmitting their performance logs back to the central database.5

The teacher dashboard is structured across seven specification screens to support this ecosystem 5:

| Screen Name | Core Administrative Functionality |
| :---- | :---- |
| **Dashboard** | Displays the main teacher login dialog, local database encryption choices, and general system notifications.5 |
| **Manage Pupils** | Handles class roster construction, group definitions, single pupil creation, password assignment, and CSV imports.5 |
| **Connect Devices** | Manages local TCP/IP networking, discovery, authentication handshakes, and student device synchronization.5 |
| **Follow Activities Results** | Displays real-time student interaction logs, indicating correct answers, failed attempts, and time-on-task metrics.5 |
| **Manage Datasets** | Features specific activity configuration tools to define goals, instructions, and levels before deploying them to clients.5 |
| **Display Results as Charts** | Generates structured, graphical data representations to assess student progression across the classroom.5 |
| **Settings** | Hosts the network server configurations, including designated server IDs, operational ports, and routing variables.5 |

## **Keyboard and Shortcut Configurations**

While GCompris is primarily designed for pointer systems, many activities are optimized with custom keyboard controls to support accessibility and alternative input methods.4 These inputs bypass standard configuration files and provide immediate UI actions.10

### **Universal Navigation Shortcuts**

The following hotkeys control overall environment execution and display properties:

| Shortcut | Functional Action |
| :---- | :---- |
| Esc or Ctrl+W | Quits the currently active activity or configuration dialog and returns to the main menu.4 |
| Ctrl+Q | Immediately terminates the GCompris application, bypassing confirmation dialogs.1 |
| Ctrl+F | Toggles display rendering between windowed and fullscreen modes.4 |
| Ctrl+M | Toggles system sound between active and muted states.4 |
| Ctrl+B | Toggles the visibility of the control bar at the bottom of the viewport.4 |
| Ctrl+S | Shows or hides the primary category selection menu on the left side of the main window.20 |

### **Contextual Gameplay Key Bindings**

When navigating activities without a pointing device, the internal focus handles specific keyboard inputs depending on the active game logic 4:

* **Navigation and Selection:** Arrow keys navigate between options, cards, or active tiles.20 The Space or Enter keys validate input selections, confirm responses, or flip interactive cards.4  
* **Text and Numerics:** Number keys allow the manual typing of numerical solutions.20 The Backspace key deletes the last entered digit or character, and the Enter key submits the solution for verification.20  
* **Special Auditory Prompts:** The Tab key triggers a question repetition audio cue, mapping to the visual lips icon.4  
* **Token Dropping:** In board games like Align Four, the Left Arrow and Right Arrow keys move tokens horizontally, while the Space or Down Arrow keys drop the token into the target column.20

#### **Works cited**

1. GCompris Administration Handbook Guide | PDF | Icon (Computing) | Graphical User Interfaces \- Scribd, accessed June 28, 2026, [https://www.scribd.com/document/978743872/g-Compris](https://www.scribd.com/document/978743872/g-Compris)  
2. Chapter 1\. Administration Handbook \- GCompris, accessed June 28, 2026, [https://gcompris.net/docbook/stable6/en/administration-handbook.html](https://gcompris.net/docbook/stable6/en/administration-handbook.html)  
3. Schools \- GCompris, accessed June 28, 2026, [https://gcompris.net/schools-en.html](https://gcompris.net/schools-en.html)  
4. The User Interface \- GCompris, accessed June 28, 2026, [https://gcompris.net/docbook/stable6/en/user-interface.html](https://gcompris.net/docbook/stable6/en/user-interface.html)  
5. The GCompris Teachers Administration Handbook \- KDE Documentation \-, accessed June 28, 2026, [https://docs.kde.org/stable\_kf6/en/gcompris-teachers-handbook/gcompris-teachers-handbook/gcompris-teachers-handbook.pdf](https://docs.kde.org/stable_kf6/en/gcompris-teachers-handbook/gcompris-teachers-handbook/gcompris-teachers-handbook.pdf)  
6. Habari \- GCompris, accessed June 28, 2026, [https://gcompris.net/news/2018-12-20-sw.html](https://gcompris.net/news/2018-12-20-sw.html)  
7. Console Switches \- GCompris, accessed June 28, 2026, [https://gcompris.net/docbook/stable5/en/console-switches.html](https://gcompris.net/docbook/stable5/en/console-switches.html)  
8. Console Parameters \- GCompris, accessed June 28, 2026, [https://docs.kde.org/stable\_kf6/en/gcompris/gcompris/console-switches.html](https://docs.kde.org/stable_kf6/en/gcompris/gcompris/console-switches.html)  
9. gcompris \- Ubuntu Manpages, accessed June 28, 2026, [https://manpages.ubuntu.com/manpages/trusty/man6/gcompris.6.html](https://manpages.ubuntu.com/manpages/trusty/man6/gcompris.6.html)  
10. GCompris Educational Software Guide | PDF | Computer File \- Scribd, accessed June 28, 2026, [https://www.scribd.com/document/265096134/Manual-GCompris](https://www.scribd.com/document/265096134/Manual-GCompris)  
11. GCompris a high quality educational software for children \- Ubunlog, accessed June 28, 2026, [https://en.ubunlog.com/gcompris-a-high-quality-educational-software-for-children/](https://en.ubunlog.com/gcompris-a-high-quality-educational-software-for-children/)  
12. Lesson 6: macOS Preferences \- Jamf, accessed June 28, 2026, [https://learn.jamf.com/r/en-US/jamf-100-course-current/Lesson\_6](https://learn.jamf.com/r/en-US/jamf-100-course-current/Lesson_6)  
13. Why does macos use binary config files like plist files? Why not be closer to the unix-style of .config (or dotfiles as they are known)? \- Reddit, accessed June 28, 2026, [https://www.reddit.com/r/MacOS/comments/1l2tyug/why\_does\_macos\_use\_binary\_config\_files\_like\_plist/](https://www.reddit.com/r/MacOS/comments/1l2tyug/why_does_macos_use_binary_config_files_like_plist/)  
14. Download \- GCompris, accessed June 28, 2026, [https://gcompris.net/downloads-en.html](https://gcompris.net/downloads-en.html)  
15. Last ned \- GCompris, accessed June 28, 2026, [https://gcompris.net/downloads-nn.html](https://gcompris.net/downloads-nn.html)  
16. News \- GCompris, accessed June 28, 2026, [https://gcompris.net/newsall-hi.html](https://gcompris.net/newsall-hi.html)  
17. Frequently Asked Questions \- GCompris, accessed June 28, 2026, [https://gcompris.net/faq-gl.html](https://gcompris.net/faq-gl.html)  
18. The GCompris Administration Handbook, accessed June 28, 2026, [https://gcompris.net/docbook/stable6/en/index.html](https://gcompris.net/docbook/stable6/en/index.html)  
19. Customizing activities \- GCompris, accessed June 28, 2026, [https://gcompris.net/docbook/stable6/en/customizing-activities.html](https://gcompris.net/docbook/stable6/en/customizing-activities.html)  
20. Screenshots \- GCompris, accessed June 28, 2026, [https://gcompris.net/screenshots-en.html](https://gcompris.net/screenshots-en.html)  
21. Schools \- GCompris, accessed June 28, 2026, [https://gcompris.net/schools-ga.html](https://gcompris.net/schools-ga.html)  
22. GCompris, KDE's collection of educational activities, publishes version 26.0 \- Reddit, accessed June 28, 2026, [https://www.reddit.com/r/kde/comments/1qvlh63/gcompris\_kdes\_collection\_of\_educational/](https://www.reddit.com/r/kde/comments/1qvlh63/gcompris_kdes_collection_of_educational/)  
23. Chapter 2\. Specifications, accessed June 28, 2026, [https://docs.kde.org/stable\_kf6/en/gcompris-teachers-handbook/gcompris-teachers-handbook/specifications.html](https://docs.kde.org/stable_kf6/en/gcompris-teachers-handbook/gcompris-teachers-handbook/specifications.html)

[image1]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABcAAAAWCAYAAAArdgcFAAAA7klEQVR4Xu2UzwoBURTGPwsvIMXazmN4Axvv4E+E7LyAhdeQhY1Slt5BUTayIBELWxY4956ZmjmdS3cnza++pvmd09c0cxsg4RfoUWpSfqFE2VBelJGYYUx5gIcm9fj4I13KM3JfBXeo+Jab/aLiBsJZfMrL0J/yDt17lS+gl+yge6/yG/SSNXRvZUNKB2ZXK1lC91Y2pXRwhF6ygu6tbEnpwPXOt9C9lW0pHfShl3w8LR0pAyqUnHBmP6O4mXDIBoOhHBAp6B/wBD56IXnwTjoUE8qFcqDsg+sZ/EuIMgX/eyRX8P4cXFyIjxP+gjfWtkfKl0PMlgAAAABJRU5ErkJggg==>

[image2]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABQAAAAaCAYAAAC3g3x9AAAA/ElEQVR4XmNgGAXUApuB+D8JmCAAKQrDIoauWQOLGAYQYoC4EBkwMUA0XkATB4FH6ALoYCsQM6KJFTBADPRHE2cD4j40MQyQjy4ABO8ZsHtNAIjF0QWJAdjCj2zAzAAx7Ay6BLmgnAFioDe6BBQoAvELIP4AxJJocljBZwb83v2JxManDg7whV8vEC9G4h8G4ngkPgYAJQt84feUAWIoDKxlgCQ7nGA2A8TABDRxGPgNxJ1I/JVAfAOJDwZBQPyNAZL23kIxKBx/MWB6/TYQ9yDxQS7chcQnGSxiwAzDJiQ+yYALiN8g8f8isckGxUB8CYjvAnEGmtwoGBEAAMzaQ3YAzmB6AAAAAElFTkSuQmCC>

[image3]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAA8AAAAaCAYAAABozQZiAAAAlklEQVR4XmNgGLlgMxD/JwGjAJBAGBYxdIUa6GJCDBCbkQETA0TRBTRxEHiEzNkKxIzIAkBQwADR7I8mzgbEfcgC+cgcKHjPgOlkEBAAYnF0QXSAzb9EAWYGiMYz6BLEgHIGiGZvdAliwGcGMp0MAmT7FxQVZPt3NgNEcwKaOE4QBMTfGCBx+xaKQf7+xUCm80fBKIADAO8/LWwyw7tTAAAAAElFTkSuQmCC>

[image4]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABEAAAAZCAYAAADXPsWXAAAAxklEQVR4XmNgGAWEQAsQfwTi/1D8HYjfA/EHIP4LFXsGV00AwAxBB1IMEPEv6BLYAEjhJnRBKMBlAQrwY4AoMkCXAAJBBoQ38YKzDLhtgrmCCV0CHcAUKkOxBhD3Q8VWIqnDC0CK9wGxCxA7Q+k4qPhWJHU4ASw8DNElgICdASJ3F10CHZxnwB0eIEBUzIAUfEYXhIIUBoj8UXQJZABLSOXoEkBgxACR+40uAQOgwDvJgHDqDSA+CMR7oTRMfAJMwygYBQMGAM+MOSX549GiAAAAAElFTkSuQmCC>

[image5]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABYAAAAaCAYAAACzdqxAAAABC0lEQVR4XmNgGAX0BvVA/AWI/0PxWVRpDPCQAaEWpK8YVRoTwBSDMC6gB8S1DBA1xmhyOMETBoTLcYHHQHyMAb8aFOAFxClAvIUBt6Z1UJqQr1DACSgNCi9smniAOBfKBsmvRpLDC2CGgcINxJZBkgOBH1DanQEir4Ukhxc8RWKDNMYh8fOBmBvKBvkMm4+wApAr0pD4II0LkfjI3iYrfGEApBEU+yDwDFmCASK3Ck0MJ0B3AcxVtkCsgyTuDRXXRhLDC16i8d8wQAy4jSYOypHojsAKGIH4LgMkiyKD5QzYDSAqfHuA+AMQvwXiz0D8B0nOB4hDkfhfGRBqPwHxbyCuRJIfBaNgFBADAG1ESZl9s8ZBAAAAAElFTkSuQmCC>

[image6]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAwAAAAbCAYAAABIpm7EAAAAiklEQVR4XmNgGAVDGnwH4k9AfBjKfwrEJ4D4PxCfhimCgTQgVgdiGwaIgn9IcspQMRTwB0pPZ4BIMiLJaUHFsAKQxC80sfVQcawAJHEMTewjVBwrAEl4YBGrRBMDg3AGTJNCkcSEgHgKkhzDZQZMDVuQxF4gS4AAKKQmo4mxMkA0gLAAmtwoGGoAAFUtI0LZEJOuAAAAAElFTkSuQmCC>

[image7]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAmwAAABECAYAAAA89WlXAAAErUlEQVR4Xu3dWah9UxwH8GWIMiTeJJQhSWR6kJR/yBCZxQtPlAhlKPGIULwoQyljyVQUXngh85ShPAhFocwzma1fe+3+y7LPufvc7rn3/PX51Ldz9m/t81/rf+7D/bWnmxIAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAALBi9su5dUpuyrkm59D+A3N0evrv/HVuzrkqZ9f+A6so5nwj580qr+e8nPNAzg7rd52711I3f6xn52asdn/OWyUvlVqs95acZ8v20+UVAFhgv+e8mrMuZ9+cv0sOzNkj57KyfWbZf55ingdzDkndWu4qtb1z9sw5vmzvWPZfCzH/M00tGqior6aHUjfnRe1AsX3Oe6nbJ36Odf2TavuF6j0AsKDaRiO2/2pqH+Zs1dRW2uY53za1r1N3dKjWrne1xfwHtMXU1e9ui3MUP6OY87F2oPghdc340PdV14bGAYAF8kTORk0tfoGf0NT+bLbn4Zu2kLq1bDtQm0UcUdq4LVaOaQtTxFomzR/1dW2xsU9bqJzXFpbwcermHFrP/uU1xh6uB7Ldcn4s709M3WnVz9cPAwCLpj2dFo3NUANwaVuYg3Ob7bi2bmgtF7aFEYb+nfBbzhZtcYobU3fUrxUN0KQ5au/mHNwWU3cdWdskT3N0ztmpO4o2NO9HOUembmyvZuzqnNPK+01yvq/GAIANwCVpuAFYC3GqbyXX8kizHacLZz3NG0car03dDQi7p66RjTU+X++0hPfTv4/EbZlzSrU9xovldeg76v+fsU87BgD8D3yXugvoZxGn+e4dyD2pu6brzpw7cm5P3em4sSad7luuK3IeLe+jWdu6Ghsr1nNUzuElJ+X8kfNOvdMIH5TXaNbi87Pqv5eLq/chGtALyvuV/v4AgAURv+C3aYtrYNPUrSXuXm2dkXNdef9UPTDScpuYOGU56bPLaY7ipoEb2uII0ZTdV97319QdVravL68h6rNeFwcALLiD0uxNx7xcnrq1HNsOpH+vcdb1RpMUDc/j7cAIcbfqpPlmbdi2yzk15+2cI5qxpURTVj+mI+aN69J2qWp9vb2hBADYwD2ZZms6eufk/Doy8Ry1MaZdxF/Xf8o5q9qepm/WQtzg0J8eHWtSUxYX9Ud96GaEIdGsxSnZXjRts2hPocbccQ1dPX8cJR1aKwCwAYsjMZMaktW2WZq+lrr+Wc5t1fYkkx5LEk1cnH5dSjwmI+aNhwjXPi31sQ8VfiXnuLaYuqN38bDgpQw1YrEdzXAt7kZdzlFEAGABnZzzc+qOznyZugfXxqMu2qZgNcSfwIojZl+l9Wv5paRWry32jZsJpok7OqMJnGSpZiu+j2j4+iYyEo1eHCU7v9pvjHVtoRJ/cmua+D7i5xQ3hsSa+of3Ppe6hw6HL8p47BdHKWPdcWMDAMCqqhu2aJz6C+4BAFgQdcO2FkcCAQAYYaecK9siAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAsDz/ABlZ/ok8n6yBAAAAAElFTkSuQmCC>