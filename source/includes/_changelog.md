# Changelog

## 2.2.0
### Added
- Populate device information semi-automatically
- Recognize match and populate device properties
- Ability to detect and populate automatically new capabilities in device information

### Changed
- @WhenSuccess and @WhenFailed and better parsing response in some corner cases.
- @Device now support device properties

## 2.1.0
### Added
- Populate capabilities based on device inheritance
- Proper copy and override methods
- Some hand-writer imports
- Detect and implement properly inheritance of devices

### Changed
- Better documented generated files
- Better validation of annotated classes
- Refactoring and restructure of annotated elements.
- Refactoring when coping methods on generated target class
- Error messages when arguments does not match method params.
- `CommandAnnotatedElement` improvements
- `DeviceAnnotatedElement` improvements
### Fixed
- Execution regardless classpath

## 2.0.0
### Fixed
- Issues retrieving parameters name
