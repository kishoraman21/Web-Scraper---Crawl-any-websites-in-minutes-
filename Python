from flask import Flask, render_template, request, send_file
from bs4 import BeautifulSoup
import requests
import re
import pandas as pd
from io import BytesIO

app = Flask(__name__)

@app.route("/", methods=["GET", "POST"])
def index():
    global result
    result = None
    if request.method == "POST":
        url = request.form.get("url")
        option = request.form.get("option")
        css_selector = request.form.get("css_selector")
        regex_pattern = request.form.get("regex_pattern")
        
        # print(regex_pattern)
        
        response = requests.get(url)
        soup = BeautifulSoup(response.text, 'html.parser')
        
        if option == "links":
            result = [a['href'] for a in soup.find_all('a', href=True)]
        elif option == "emails":
            result = re.findall(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b', response.text)
        elif option == "p_tag":
            result = [p.get_text() for p in soup.find_all('p')]
        elif option == "text":
            result = [element.get_text(strip=True) for element in soup.select(css_selector)]
        elif option == "regex":
            result = re.findall(regex_pattern, response.text)
        
    return render_template("index.html", result=result)

@app.route("/download/<file_type>")
def download_file(file_type):
    if not result:
        return "No data to download", 400
    
    df = pd.DataFrame(result, columns=["Data"])
    
    if file_type == "csv":
        output = BytesIO()
        df.to_csv(output, index=False)
        output.seek(0)
        return send_file(output, mimetype="text/csv", as_attachment=True, download_name="scraped_data.csv")
    
    elif file_type == "xlsx":
        output = BytesIO()
        with pd.ExcelWriter(output, engine='xlsxwriter') as writer:
            df.to_excel(writer, index=False)
        output.seek(0)
        return send_file(output, mimetype="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", as_attachment=True, download_name="scraped_data.xlsx")

if __name__ == "__main__":
    app.run(debug=False)
