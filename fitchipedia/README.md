Fitchipedia
===========

Extensions
----------

### MultiBoilerplate
```
Html::submitButton( wfMessage( 'multiboilerplate-submit' )->plain() );
```
Line 138 Hooks.php

Maintenance
-----------
```
docker-compose exec mediawiki php maintenance/run.php update
```

Clear bogus users:

```
dc exec mediawiki php maintenance/run.php removeUnusedAccounts.php --ignore-touched 0 [--delete]
```
