# Version 1.1.1 (2020-10-06)

## Bug Fixes
* Fixed issue with removing resources during cleanup
* Fixed issue where during retry it kept appending the same network to
  the network list during the find ip availability if that setting was set
* Fixed issue where setting the ssh key private file permission was failing

# Version 1.1.0 (2020-08-24)

## New features
* Support for add/remove type openstack actions
* Support for adding a hash to asset name to create unique name

## Enhancements
*  Provide  an abstraction of 'max' for non server resources as well

## Bug Fixes
* Fixed for issue where the schema was failing to validate cloud_file key
* Fixed for issue where during retries we attempt to do an initial cleanup
  of an asset in case error is not transient and an asset exists in a bad state
* Update valid_creds_combo method in th osp_schema_extension to allow correct validation

## Doc Changes
* Included how to add hash to asset name to get a unique name


# Version 1.0.0 (2020-07-24)

## New features
* Added retry logic. A user can define in the teflo.cfg 
  the number of retry attempts and how long to wait between
  attempts when errors during create/delete are encountered.

## Enhancements
* None

## Bug Fixes
* None

## Doc Changes
* Included how to enable the retry logic in the teflo.cfg

## Test/CI Enhancements
* None


# Version 0.2.0 (2020-07-13)

## New features
* None

## Enhancements
* Support for provisioner configuration options in teflo.cfg to be able to  
  support defining a public_network when a node is connected to two networks
  with similar names. Support the ability for the provisioner to select and 
  filter the best network based on ip availability.

## Bug Fixes
* None

## Doc Changes
* Added Users Doc to show how use the plugin

## Test/CI Enhancements
* Added missing .bumpversion.cfg file to be able to increment versions.


# Version 0.1.0 (2020-07-06)

## New features
* First initial release of the openstack client plugin
* Supports all openstack client command line functionality

## Enhancements
* None

## Bug Fixes
* None

## Doc Changes
* None

## Test/CI Enhancements
* None
