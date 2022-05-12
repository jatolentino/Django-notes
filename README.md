# Deploy in Django
### 1 Create & activate the virtual enviroment
- In MINGW
	```bash
	python -m venv enviroment
	source env/Scripts/activate
	```
- In VSCODE
	```bash
	python -m venv enviroment
	env/Scripts/activate
	```
### 2 
### 3
### 4
### 5
### 6
### 7
### 8
### 9
### 10
### 11
### 12
### 13
### 14
### 15
### 16
### 17
### 18
### 19 Urls in the app, namespaces
- Create & edit leads/urls.py
  ```python
  from django.urls import path
  from .views import home_page

  app_name = "leads"
  urlpatterns = [
    path('all/', home_page)  OR path('', home_page)
  ]
  ```

- Edit crm/urls.py
delete -> from leads.views import home_page
delete -> path('', home_page)
	```python
	from django.urls import path, include

	urlpatterns = [
		path('admin/', admin.site.urls),
		path('leads/', include('leads.urls', namespace="leads"))
	]
	```

Test: navigate to `https://127.0.0.1:8000/leads/all` OR `https://127.0.0.1:8000/leads/`

### 20 Lead's list
- Change home_page.html name to lead_list.html (leads/templates/leads/lead_list.html)
	```python
	def lead_list(request):
		leads = Lead.objects.all()
		context = {
			"leads": leads
		}
		return render(request, "leads/lead_list.html", context)
	```
- Edit leads/urls.py
	```python
	from django.urls import path
	from .views import lead_list
	app_name = "leads"
	urlpatterns = [
		path('', lead_list)
	]
	```
- Add style to lead_list.html
	```html
	<title>Document</title>
	<style>
		.lead {
			padding-top: 10px;
			padding-bottom: 10px;
			padding-left: 6px;
			padding-right: 6px;
			margin-top: 10px;
			background-color: #f6f6f6;
			width: 100%;
		}
	</style>
	
	<body>
		<h1> This is all of our leads</h1>
		{% for lead in leads %}
			<div class="lead">
				{{ lead.first_name }} {{ lead.last_name }}. Age: {{ lead.age }}
			</div>
		{% endfor %}
	</body>	
	```
Test in: `https://127.0.0.1:8000/leads/

- Create another view for the details of the leads in leads/views.py
	```python
	def lead_detail(request, pk):
		print(pk)
		lead = Lead.objects.get(id=pk)
		return HttpResponse("here is the detail view")
	```
- Import lead_detail in lead/urls.py
	```python
	from .views import lead_list, lead_detail
	
	app_name = "leads"
	
	urlspatterns = [
		path('', lead_list),
		path('<int:pk>', lead_detail)  #this is going to add a path according to the ID (primary key) of the user (pk)
	```
- Add links to the lead in leads/templates/leads/lead_list.html
	```html
	<body>
		<h1> This is all of our lead lead</h1>
		{% for lead in leads %}
			<div class="lead">
				<a href="/leads/{{ lead.pk }}/"> {{ lead.first_name }} {{ lead.last_name }}</a>. Age: {{ lead.age }}
			</div>
		{% endfor %}
	</body>
	```
Test: `https://127.0.0.1:8000/leads/1/`
- Modify the lead_detail, in leads/views.py
	```python
	def lead_detail(request, pk):
		lead = Lead.objects.get(id=pk)
		context = {
			"lead": lead
		}
		return render(request, "leads/lead_detail.html", context)
	```
- Create & edit the html file templates/leads/lead_details.html
	```html
	<!DOCTYPE html>
	<html lang="end">
	<head>
		<meta charset="UTF-8">
		<meta name="viewport" content="width=device-width, initial-scale=1.0">
		<title>Document</title>
		<style>
			.lead {
				padding-top: 10px;
				padding-bottom: 10px;
				padding-left: 6px;
				padding-right: 6px;
				margin-top: 10px;
				background-color: #f6f6f6;
				width: 100%;
			}
		</style>
	</head>
	<body>
		<a href="/leads">Go back to leads</a>
		<hr />
		<h1> This is the details of {{ lead.first_name }}</h1>
		<p> This persons age: {{ lead.age }} </p>
	</body>
	</html>
	```
### 21  create leads with Forms
- Create the lead_create in leads/views.py
	```python
	def lead_create(request):
		return render(request, "leads/lead_create.html")

- Create the html file: templates/leads/lead_create.html
	```html
	<!DOCTYPE html>
	<html lang="end">
	<head>
		<meta charset="UTF-8">
		<meta name="viewport" content="width=device-width, initial-scale=1.0">
		<title>Document</title>
	</head>
	<body>
		<a href="/leads">Go back to leads</a>
		<hr />
		<h1> Create a new lead </h1>
	</body>
	</html>
	```
- Edit the PATH leads/urls.py
	```python
	from .views import lead_list, lead_detail, lead_create
	
	app_name = "leads"
	urlpatterns = [
		path('', lead_list),
		path('<int:pk>', lead_detail),
		path('create', lead_create),
	]
	```
Test: `https://127.0.0.1/leads/create`

- Create & edit the file forms in templates/leads/forms.py
	```github
	from django import forms
	
	class Leadform(forms.Form):
		first_name = forms.CharField()
		last_name = forms.CharField()
		age=forms.IntegerField(min_values=0)
	```
- Import Leadform to leads/views.py
	```python
	from .forms import LeadForm
	
	def lead_create(request):
		context = {
			"form": LeadForm()
		}
		return render(request, "leads/lead_create.html", context)
	```
- Configure the form html and add security to the form: leads/lead_create.html
	```html
	<body>
		<a href="/leads"> Go back to leads</a>
		<hr />
		<h1> Create a new lead</h1>
		<form method="post"> <!-- form method="post" action="/leads/another-url/"> -->
			{% csrf_token %}
			{{ form.as_p }}
			<button type="submit" >Submit</button>
		</form>
	</body>
	```
- Grab the data from the POST form: leads/views.py
	```python
	from .models import Lead, Agent
	def lead_create(request):
		form = LeadForm()
		if request.method == "POST":
			print('Receiving a post request')
			form = Leadform(request.POST)
			if form.is_valid():
				print("the form in valid"P)
				print(form.cleaned_data)
				first_name = form.cleaned_data['first_name']
				last_name = form.cleaned_data['last_name']
				age = form.cleaned_nadata['age']
				agent = Agent.objects.first()
				Lead.objects.create(
					first_name=first_name,
					last_name=last_name,
					age=age,
					agent=agent
				)
				print("The lead has been created")
		
		context = {
			"form": LeadForm()
		}
		return render(request, "leads/lead_create.html", context)
	```
- Test: Go to `https://127.0.0.1:8000/leads/create/` and create a lead, then SUBMIT <br>
	Verify in `https://127.0.0.1:8000/leads/` <br>
	Verify the prompt in the VS code or check in `https://127.0.0.1:8000/admin/leads/lead/`
	
- Redirect the create page to another tab
	In leads/view.py
	```python
	from django.shortcuts import render, redirect
	:
	:
	agent = Agent.objects.first()
		Lead.objects.create(
			first_name=first_name,
			last_name=last_name,
			age=age,
			agent=agent
			)
			return redirect("/leads")
	: 
	```
### 22 Using Django ModelsForm
















































