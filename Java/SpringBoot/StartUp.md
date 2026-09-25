## The Core Folders (src) 
- This is where 99% of your work happens.
- src/main/java: The "Home" of your code. Your Java classes, controllers, and logic live here. They are usually organized by "packages" (e.g., com.example.project).
- src/main/resources: Non-java files.
- static/: Where you put CSS, JavaScript, or images if you are building a front-end within Java.
- templates/: Where HTML files (Thymeleaf/FreeMarker) live.
- application.properties (or .yml): This is the most important config file. It's where you tell the app which database to use or what port to run on (e.g., server.port=8081).
- src/test/java: Where your unit tests and integration tests live. Professional projects always have code here to ensure things don't break. 

## The Project Setup Files 
- These files sit in the "Root" (the very bottom of the list) and manage how the project is built.
  - pom.xml (Maven) OR build.gradle (Gradle): This is your Project Manifesto. It lists every library (dependency) your project uses. If you want to add a "Security" or "Database" feature, you add it here.
  - .gitignore: Tells Git which files to ignore (like your personal IDE settings or compiled code) so you don't upload "trash" to GitHub.
  - mvnw / mvnw.cmd: These are "Maven Wrappers." They allow anyone to run your project even if they don't have Maven installed on their computer. 

## Hidden IDE Folders 
- .idea/: These are internal settings for IntelliJ IDEA. You should generally never touch these files manually.
- target/ (or build/): This folder is created when you "Run" the app. It contains the compiled .class files. If your project acts weird, you can delete this folder and "Rebuild" to start fresh. 

## Key Java Files to Look For 
- YourProjectApplication.java: **Marked with @SpringBootApplication.** This is the entry point. When you hit "Run," this is the file that actually starts the engine.
- Controllers (...Controller.java): These handle the URLs (like /home or /login).
- Repositories (...Repository.java): These handle talking to your database. 


# Error Faced
- Identity column type must be smallint, integer, or bigint
  - This error happens because PostgreSQL IDENTITY columns (e.g., auto-incrementing numbers) only support integer types (smallint, integer, or bigint).
 
# Postgres
- \d users
  - List schema of users table
- \l
  - list all dbs
- \dt
  - list all tables
- \c <other_database_name>
  - Switch to other database 
