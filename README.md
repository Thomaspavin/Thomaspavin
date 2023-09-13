- 👋 Hi, I’m @Thomaspavin
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
Thomaspavin/Thomaspavin is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
SCRIPT(Artefact)

#!/bin/bash
#Syntax : ./script.sh domain_name
#Example : ./script.sh google.com

if [ $# -eq 0 ]; then
 echo -e "No arguments provided"
 echo -e "\n #Syntax : ./script.sh domain_name "
exit 1
fi

mkdir /home/kali/Project/$1
cd /home/kali/Project/$1

echo -e "Whois\n"

ip_a=$(dig +short $1)
echo -e "IP Address: $ip_a"
whois $ip_a | tee whois.txt

echo -e "Subdomain Enumeration\n"

assetfinder --subs-only $1 | tee $1.asset.txt
amass enum --passive  -d $1 | tee $1.amass.txt

echo -e " Total Subdomain List \n"
cat $1.asset.txt  $1.amass.txt | sort | uniq | tee $1.sub.txt

echo -e "\n----------------------------------------------------"
echo -e  "----------------------------------------------------\n"

echo -e " Checking Live Hosts \n"

httpx-toolkit -list $1.sub.txt -timeout -100 -o $1.txt


echo -e "\n----------------------------------------------------"
echo -e  "----------------------------------------------------\n"

echo -e "Nmap Scan"

nmap -A $ip_a  -oN nmap.txt 

echo -e "\n----------------------------------------------------"
echo -e  "----------------------------------------------------\n"

echo -e "DIRSEARCH \n"

dirsearch -l  /home/kali/Project/$1/$1.txt -o /home/kali/Project/$1/$1.dir.txt

echo -e "\n----------------------------------------------------"
echo -e  "----------------------------------------------------\n"

echo -e " NUCLEI SCAN \n\n"

echo -e "\n----------------------------------------------------"
echo -e  "----------------------------------------------------\n"

echo " Checking for CVE ";

cat $1.txt | nuclei -t  /home/kali/.local/nuclei-templates/cves -o cve.txt

echo -e "\n----------------------------------------------------"
echo -e  "----------------------------------------------------\n"

echo "Checking for Subdomain  takeovers ";

cat $1.txt | nuclei -t  /home/kali/.local/nuclei-templates/takeovers  -o takeover.txt


echo -e "\n----------------------------------------------------"
echo -e  "----------------------------------------------------\n"

echo "Checking for Exposures ";

cat $1.txt | nuclei -t  /home/kali/.local/nuclei-templates/exposures  -o exposure.txt


echo -e "\n----------------------------------------------------"
echo -e "----------------------------------------------------\n"


echo "Checking for misconfiguration ";

cat $1.txt | nuclei -t  /home/kali/.local/nuclei-templates/misconfiguration  -o misconfig.txt

echo -e "\n----------------------------------------------------\n"
echo -e "\n----------------------------------------------------\n"

echo -e "Taking Screenshots of Web Assets"

eyewitness -f $1.sub.txt -d screenshot --web --no-prompt --timeout 50 

echo -e "\n\n----------------------------------------------------\n"
echo -e "\n\n----------------------------------------------------\n"

theHarvester -d $1  -b google,twitter | tee harvester.txt


