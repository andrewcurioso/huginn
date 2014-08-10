# Hugann Scenario Package Specification
Draft 0.1

This is a proposed Hugann scenario specification for discussion and improvement by the community.

## Design Considerations

- **Security**: The origin of the package must be verifiable
- **Portability**: The package must be able to be built and extracted on all platforms that support Hugann
- **Usability**: It must be easy to create and install packages
- **Extensible**: It must be able to reasonable support Hugann into the future and be easily versioned to support minimum system requirements

## Scenario Package Format

### Contents
- **manifest.json** contains the meta information about the package
- **agents.json** contains the various agent configurations
- **icon-128.png** contains a 128x128px icon for the extension
- **icon-16.png** contains an optional 16x16px icon for the extension
- **README.md** contains an instructional file for the package in markdown format
- **CHANGES.md** contains a list of changes by version; may be displayed during upgrades

### Manifest
The manifest file consists of Json data about the scenario.

The fields include:

Name | Type | Description
------------ | ------------- | -------------
name | string | The name of the scenario
description | string | A short description for the scenario - longer descriptions belong in README.md
version | string | A version number for the scenario in the format of major.minor.build - Example 1.0.10
requirements | object | Requirements to use the scenario
requirements.hugann_version | string | The required version of Hugann
requirements.gems | object | A object containing key/value pairs where the key is the Gem name and the value is the version requirements
requirements.supported_os | array | An array of supported operating systems with wildcards allowed - `[ "*" ]` for all operating systems
requirements.executables | object | An an object containing key/value pairs for required executables where the key is the executable name and the value is the version - the implementation may be operating system specific
guid | string | A globally unique identifier for the scenario generated when the scenario is first created (persists across versions)
exported_at | string | The date the scenario was packaged - in the format in ISO 8601 format
author | string | The author name
urls | object | Relevant URLs associated with the scenario
urls.homepage | string | The homepage for the author/scenario
urls.source | string | The URL where the source code for the scenario can be found
urls.update | string | The update URL for the scenario
parameters | object | Parameters to customize the scenario

### Agents
Agents consist of a Json array consistent consisting of objects with the following fields:

Name | Type | Description
------------ | ------------- | -------------
type | string | The type of agent
name | string | The name for this agent
disabled | boolean |  A boolean `true` (for disabled) or `false` (for enabled)
guid | string | A globally unique identifier for this agent
options | object | The configuration for the agent
schedule | string | The frequency or time for the agent to run
keep_events_for | integer | How many days to keep events for - `0` for indefinitely

### Package File Format
Packages have the file extension *.huginnscenario

The package file format consists of a Zip file with additional header fields added to the front. The additional headers provide version information for the package as well as signing information. This format is almost identical to the Google CRX format for Chrome extensions.

Name | Type | Length | Value | Description
------------ | ------------- | ------------- | ------------- | -------------
magic number	| char[] | 64 bits | Huginn01 | A constant at the beginning of every package file
version | unsigned int | 32 bits | 1 | The version of the *.huginnscenario file format used (initially 1).
public key length | unsigned int | 32 bits | | The length of the RSA public key in bytes.
signature length | unsigned int | 32 bits | | The length of the signature in bytes.
public key | byte[] | size of key | | The contents of the author's RSA public key, formatted as an X509 SubjectPublicKeyInfo block.
signature | byte[] | size of signature | | The signature of the ZIP content using the author's private key. The signature is created using the RSA algorithm with the SHA-1 hash function.

### User Experience and Flow

#### Installing a scenario

1. User specifies the URL of a scenario or uploads a scenario file or searches for a scenerio via API to a directory website
2. The scenario file is downloaded
3. Signature is verified
    1. Proceed if the signature matches
    2. Otherwise display a stern warning
4. File is unzipped in memory
5. Manifest is read
6. The required version of Hugann is verified
    1. Proceed if the version is sufficient
    2. Otherwise halt installation
7. The required operating system is verified
    1. Proceed if the operating system and version is sufficient
    2. Otherwise halt installation
8. Required executables are verified
    1. Proceed if all required executables are installed
    2. Otherwise display the missing applications and prompt the user to contact their system administrator
9. The required gems are verified
    1. Proceed if all required gems are installed
    2. Otherwise display the missing gems and prompt the user to contact their system administrator
10. The user is prompted to enter any configuration parameters
11. The package data is copied to the database
12. The user is congratulated

#### Upgrading a scenario

1. Hugann checks for scenario updates every 24 hours
2. The user is prompted with updates to install and presented with change logs
3. The new package is downloaded
4. The signature is verified and checked against the public key from the original install
    1. Halt if the signature is invalid
    2. Halt if the public key does not match the public key for the previous version
    3. Proceed is signature and public key are valid
5. Resume from step 4 in installation procedure

### Upgrade Endpoints
The upgrade URL for a package should respond with an object containing the latest version of the scenario, the GUID, and the location of the update, and information helpful to validate the download.

Example:
````
{
  "version" : "1.2.0",
  "guid" : "abc123",
  "update_url" : "https://.../abc.huginnscenerio",
  "public_key" : "...",
  "sha1_hash" : "..."
}
```
The update URL may be over HTTPS, however, it is not necessary since the package itself is signed. In fact, serving packages over regular HTTP is recommended to facility caching.

### Include Files
The scenario may include additional files such as shell scripts. These files can be referenced from agent configurations using `scenario:path` where `path` is the name of the file relative to the root of the scenario package. Similar to the way credentials are managed.

During script execution all files attached to the scenario are unpacked to a temp filesystem. As such it is not recommended to have extremely heavy scenarios. For example: say a agent runs a shell script which runs a rake task. It would be better to abstract the heavy lifting out into a Gem and list the Gem as a requirement for the scenario than to include all the Ruby code directly in the scenario.

### Version Strings
Version strings should be in the format of `Major.Minor.Build`. For comparison purposes the same rules should be followed as for Bundler version information in Gemfiles.

When comparing version strings to a requirement the following examples should be considered:
- `1.2.0` matches only exactly version 1.2.0
- `>=1.2.0` matches any version greater than or equal to 1.2.0
- `~>1.2.3` matches any version greater than or equal to 1.2.3 but less than 1.3

### Operating System
The version string should be matched against the value of `RUBY_PLATFORM` with wildcards allowed. Matches are cases insensitive.

Examples:
- To match all versions of OSX: `*-darwin*`
- To match Linux:  `*-linux`

## References

Google Chromium CRX Format Specification - https://developer.chrome.com/extensions/crx
About CRX files - http://www.lyraphase.com/wp/projects/software/about-crx-files/

