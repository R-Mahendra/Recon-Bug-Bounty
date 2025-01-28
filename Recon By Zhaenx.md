<h1>ZHAENX</h1>
ssl.cert.sucject.CN:"target.com"

# Enumerasi Subdomain
subfinder -d target.com --all -o subdomain1.txt
assetfinder -subs-only target.com > subdomain2.txt
amass enum --passive -d target.com -o subdomain3.txt
cat subdomain1.txt subdomain2.txt subdomain3.txt | sort -u > maindomain.txt

# Filter Subdomain Aktif
cat maindomain.txt | httpx-toolkit -ports 80,443,8080,8000,8888 -threads 200 -o subdomain-alive.txt

# Crawling dan Kumpulkan URL
katana -u subdomain-alive.txt -d 5 -kf -jc -fx -ef woff,pdf,css,png,svg,jpg,jpeg,woff2,gif -o katanaUrls.txt
cat subdomain-alive.txt | hakrawler -d 5 -subs -t 50 > crawledUrls.txt
cat subdomain-alive.txt | gau > gauUrls.txt
cat katanaUrls.txt crawledUrls.txt gauUrls.txt | sort -u > allUrls.txt

# Filter URL Spesifik
grep -E "\?|\=|api" allUrls.txt > filtered-urls.txt

# Analisis Endpoint
cat allUrls.txt | grep -E ".php|.asp|.aspx|.jspx|.jsp" | grep '-' | sed 's/=.*/=/' | sort | uniq > bsqli.txt
cat allUrls.txt | Gxss | kxss | grep -oP '^URL: \K\S+' | sed 's/=.*/=/' | sort -u > xss.txt
cat allUrls.txt | gf lfi | sed 's/=.*/=/' | sort -u > lfi.txt
cat allUrls.txt | grep -E "\.js$" >> js.txt

# Analisis JavaScript
cat js.txt | while read url; do python3 linkfinder.py -i $url -o results/js-endpoints/$(echo $url | sed 's/https\?:\/\///').html done

# Directory Brute Force
cat subdomain-alive.txt | while read url; do dirsearch -u $url -e php,html,json -w /path/to/wordlist.txt -o results/$url-endpoints.txt done

# Menggunakan Dirsearch untuk brute-forcing direktori dengan beberapa status kode HTTP yang diinginkan
dirsearch -l subdomain-alive.txt -x 500,502,429,404,200 -R 5 --random-agent -t 100 -o dirsearch_result.txt -w /path/to/wordlist.txt

# Menggunakan Dirsearch dengan beberapa ekstensi file untuk menemukan direktori yang rentan
dirsearch -e conf,config,bak,backup,swp,old,db,sql,asp,aspx,aspx~,asp~,py,py~,rb,rb~,php,php~,bak,bkp,cache,cgi,conf,csv,html,inc,jar,js,json,jsp,jsp~,lock,log,rar,old,sql,sql.gz,sql.zip,sql.tar.gz,sql~,swp,swp~,tar,tar.bz2,tar.gz,txt,wadl,zip,log,xml,js,json -l subdomain-alive.txt -R 5 --random-agent -t 100 -o dirsearch_result_extended.txt

# Analisis Kerentanan
nuclei -l subdomain-alive.txt -t /path/to/nuclei-templates/ -o nucleiResult.txt
nuclei -l subdomain-alive.txt -o nucleiResult.txt


# Analisis Parameter
python3 paramspider.py --domain target.com -o param-urls.txt
arjun -u https://target.com/api/v1/endpoint -oT api-parameters.txt

# Screenshot
cat subdomain-alive.txt | aquatone -out screenshots

#WPscan
wpscan --url https://site.paytabs.com/ --random-user-agent --api-token s0jJJWD83hn6JoismMRB3nxkjRFXf75bPtYPgX0YuUM -e -t 10 -o wp-paytab.txt


#Nmap
nmap -iL subdomain-alive.txt -sV -T 5 -Pn --script vuln -oN nmap-vuln-scan.txt
