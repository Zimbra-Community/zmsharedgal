#!/bin/bash

echo "This utility will allow you to setup a shared GAL between the domains you choose. Please note, when you enter the domains, these domains will all share between each other, so all users on any domain will see all users on the other domains you have shared the GAL between."
echo
echo "MAKE SURE YOU HAVE YOUR LDAP PASSWORD AND HOSTNAME!"
echo "You can get that information with this command: zmlocalconfig -s |egrep \"ldap_root_password|ldap_url\""

echo
echo "Please enter the domains you wish to share the GAL between, separated by a space."
read DOMAINS
echo "Please enter the LDAP server hostname"
read LDAPHOSTNAME
echo "Please enter your LDAP password"
read LDAPPASS

echo "Checking your domains . . ."

read -a DOMARR <<<$DOMAINS

for i in ${DOMARR[@]}; do
#Checking if the domains exist. If not, inform the user and exit.
	res=`zmprov gd $i 2>&1`
	errorCheck="ERROR: account.NO_SUCH_DOMAIN (no such domain: $i)"
	if [ "$res" = "$errorCheck" ]; then
		echo "There was an error with $i. This domain does not exist. Exiting."
		exit
	fi
	echo "$i exists . . ."
	unset res
	unset errorCheck
done
unset i
for i in ${DOMARR[@]}; do
#Checking if the galsync account exists. If not, create it.
	res=`zmprov gd $i zimbraGalAccountId | grep zimbraGalAccountId`
	errorCheck="# name $i"
	GALACCID=${res#z*: }
	account="galsync@$i"
	if [ "$res" = "$errorCheck" ]; then
		echo "Galsync Account $account does not exist."
		echo "Creating $account"
		echo "Enter the hostname of the mailstore on which you would like this galsync account to reside"
		read server
        GALACCID=`zmgsautil createAccount -a $account -n InternalGAL --domain $i -s $server -t zimbra -f _InternalGAL`
		GALACCID=${GALACCID#$account*   }
		echo "forcing first sync of new gal account"
        zmgsautil forceSync -i $GALACCID -n InternalGAL
        fi
#getting the actual galsync account since zmgsautil addDataSource can't work with the account ID
account=`zmprov ga $GALACCID name`
account=${account##\#* }
echo "$account exists . . ."
#cycle through all domains in DOMARR, exclude current domain ($i) from being added as a datasource for itself
		for x in ${DOMARR[@]};
			do
				#check if datasource domain = current domain and skip it if so
				if [ "$x" = "$i" ]; then
				continue
				fi
				CLEANDOM=${x//[-.]/}
				#set search base
				IFS='.' read -ra DOMBASE <<< "$x"
					LEN=`echo ${#DOMBASE[@]}`
					SEARCHBASE="dc=${DOMBASE[0]}"
					j=1
					while (( $j < $LEN )); do
						TOADD="dc=${DOMBASE[$j]}"

						SEARCHBASE="$SEARCHBASE,$TOADD"
						j=$(($j+1))	
					done
			
			
				zmgsautil addDataSource -a $account -n $CLEANDOM --domain $i -t ldap -f _$CLEANDOM -p 1d
				echo "configuring datasource for $x"
				zmprov mds $account $CLEANDOM zimbraGalSyncLdapBindDn uid=zimbra,cn=admins,cn=zimbra zimbraGalSyncLdapBindPassword $LDAPPASS zimbraGalSyncLdapFilter '(&(mail=*)(zimbraAccountStatus=active)(!(zimbraHideInGAL=TRUE)))' zimbraGalSyncLdapSearchBase $SEARCHBASE zimbraGalSyncLdapURL ldap://$LDAPHOSTNAME:389
				zmgsautil forceSync -i $GALACCID -n $CLEANDOM
				#unset CELANDOM
				#unset LEN
				#unset SEARCHBASE
				#unset DOMBASE
				#unset account
			done
	done
echo "GalSync Datasources configured and synced."
echo "Done!"
exit
