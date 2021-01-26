---
layout: post
title:      "Project: RV-Experience on Rails/JS "
date:       2021-01-26 20:02:06 +0000
permalink:  project_rv-experience_on_rails_js
---



Today, mark one of the most important day of my journey at Flatiron School I have had thus far. It took me a long time to realize what all I have learnt is all about, and feels like everything magically turned into something more tangible. The RV-Experience, is a very basic app geared toward adventure enthusiasts who wants to build a list of their favorite equipment and also able to show them off to the world and possibly sharing their equipment with other enthusiasts. 


![superman is real](https://miro.medium.com/max/598/1*J5kbdrrNi3PkdX_ZZn8xFw.jpeghttp://)

**But how did I do it?**
First of all, I would like to talk about the structure. It is built on two different files: one for the frontend(where the users see all the magic happens) powered by a JavaScript modular structure and the other where my database stored and logics handled behind the scene, as always, powered by sweet RoR. The reason they are built separately is for future scalability and expansion of this app, in a very organized easy to follow and easy to maintain codebase.

**What requirements are being fulfilled?**
The purpose of this app is for the front end to send/receive requests to my Rails api by using fetch and axios as primary javascript instrumentations to facilitate this process. It is REQUIRED to utilize at least 3 fetch requests for this project: What it basically does is on the browser, a user(you) can create an instance of a company(one fetch), get the list of all the companies(two fetch), delete an instance of a company(three fetch). As an extra feature, I have also built creating an instance of associated record, RV, to the company and in the process of creating a filtering features in which records of each RV particularly associated with each company can be rendered on the page whenever the button clicked!

<h1>Back-end Stuff:</h1>

<h1>Controllers:</h1>
<h3>company controller:</h3>

```
class Api::V1::CompaniesController < ApplicationController
def index
companies = Company.all
render json: companies, status: 200
end
def create
company = Company.new(company_params)
if company.save
render json: {company: company}, status: 200
else
render json: {errors: “error found”}
end
end
def show
company = Company.find_by(id: params[:id])
if company
render json: company, status: 200
else
render json: {message: “No company found!”}
end
end
def destroy
company = Company.find_by(id: params[:id])
company.destroy
end
private
def company_params
params.require(:company).permit(:name, :email, :password, :address, :city, :state, :zipcode, :phonenumber, :building_number, rv_attributes: [
:name, :capacity, :rate_per_day, :image_link]
)
end
end
```

<h3>Rv Controller:</h3>
```
class Api::V1::RvsController < ApplicationController
def index
rvs = Rv.all
render json: {rvs: rvs}, status: 200
end
def new
company = Company.find(params[:id])
if company
rv = Rv.new
render json: RvSerializer.new(rvs)
else
render json: rv.errors, status: :unprocessable_entity
end
end
def create
company = Company.find(params[:company_id])
if company
rv = company.rvs.create(rv_params)
render json: company, status: 200
else
render json: rv.errors, status: :unprocessable_entity
end
end
def show
rv = Rv.find_by(id: params[:id])
options = {include: [:company]}
if rv
render json: RvSerializer.new(rv, options)
else
render json: {message: “No RV found!”}
end
end
def update
@company = Company.find_by_id(params[:company_id])
if @company && !@company.nil?
@rv = @company.rvs.find_by_id(params[:id])
@rv.update(rv_params)
render json: @rv, status: 200
else
render json: @rvs, status: 200
end
end
def destroy
@rv = Rv.find_by_id(params[:id])
@rv.destroy
end
private
def rv_params
params.require(:rv).permit(:name, :capacity, :rate_per_day, :company_id)
end
end
```

<h1>Models:</h1>
<h3>Company's Model:</h3>
```
class Company < ApplicationRecord
include ActiveModel::Serializers::JSON
has_many :rvs, dependent: :destroy
accepts_nested_attributes_for :rvs, allow_destroy: true, reject_if: proc{|attribute| attribute[‘name’].blank?}
validates :email, :name, uniqueness: true
validates :name, presence: true
end
```

<h3>RV's Model:</h3>
```
class Rv < ApplicationRecord
belongs_to :company
has_many :reservations
has_many :users, through: :reservations
end
```

<h1>Front-End Stuff:</h1>
My front-end is built on one index.html file that serves as my Single Page Application, where all these jazz summarized into this one single page without much static codes built into it. That is where my two main subdirectories with all their JavaScript files play the magic: Components and Adapters.

<h3>Components:</h3>
This is where most of the actions take places, all the functions necessary to do their jobs, manipulating the DOM, and sending requests:


Companies class: This is where most of the magic of my app take place: Where you can have use the adapters, fetch and load resources through functions, and bind eventlisteners on the page:

```
class Companies {
constructor(){
this.companies = []
this.companyId = []
this.filteredRvs = []
this.adapter = new CompaniesAdapter()
this.rvsAdapter = new RvsAdapter()
this.rvs = new Rvs()
this.rv = new Rv()
this.companyForm = document.getElementsByClassName(“new-company-form”)
this.companyFormSubmit = document.getElementById(“company-form-submit”);
this.companyDeleteBtn = document.getElementById(“delete-company”)
this.rvFormSubmit = document.getElementById(“rv-form-submit”)
this.listOfRvs = document.getElementsByClassName(“btn btn-info”)
this.fetchAndLoadCompanies()
}
fetchAndLoadCompanies(){
document.addEventListener(“DOMContentLoaded”, ()=>{
this.adapter.getCompanies()
.then(companiesData => this.createCompanies(companiesData))
.then(() => this.addCompaniesToDom())
.then(() => this.bindEventListeners())
.then(() => this.callToRemove())
})
}
async bindEventListeners(){
this.companyFormSubmit.addEventListener(“click”, function(event){this.postCompany(event);}.bind(this))
this.rvForms = document.getElementsByClassName(“btn btn-dark”)
this.rvList = document.getElementsByClassName(“btn btn-info”)
for (let showForm of this.rvForms){
showForm.addEventListener(“click”, (event)=>{
this.rv.renderRvForm(event)
})
}
for(let compRvs of this.listOfRvs){
// compRvs.addEventListener(“click”, (event)=>{this.getRvs(event);})
compRvs.addEventListener(“click”, (event)=>{this.getRvs(event)})
}
}
// POSTING TO MY RAILS API:
postCompany(event){
event.preventDefault()
const form = event.target.parentElement
const configurationObject = {
method: “POST”,
headers: {
“Content-Type”: “application/json”,
“Accept”: “application/json”
},
body: JSON.stringify({
“name”: form[0].value,
“address”: form[1].value,
“city”: form[2].value,
“state”: form[3].value,
“zipcode”:form[4].value,
“phonenumber”:form[5].value,
“building_number”:form[6].value,
“email”:form[7].value,
})
};
form.reset();
this.adapter.postCompanyToApi(configurationObject).then((json)=>{
let companyPost = new Company(json.company.id, json.company.name, json.company.address, json.company.city, json.company.state,json.company.zipcode, json.company.phonenumber, json.company.building_number, json.company.email)
companyPost.createCompanyCard()
})
}
postRv(event){
console.log(event)
}
async createCompanies(companiesData){
for(let comp of companiesData.data){
try {
this.companies.push(new Company(comp.id, comp.name, comp.address, comp.city, comp.state, comp.zipcode, comp.phonenumber, comp.building_number, comp.email, comp.rvs))
} catch(error){
console.error(error)
}
}
}
addCompaniesToDom() {
for (let company of this.companies) {
company.createCompanyCard()
}
}
callToRemove(){
document.querySelectorAll(“#delete-company”).forEach(deleteBtn => deleteBtn.addEventListener(‘click’, (event)=>{
const configObj = {
method: ‘DELETE’,
headers: {
‘Content-Type’: ‘application/json’,
‘Accept’: ‘application/json’
}
}
let companyUl = this.adapter.baseUrl + `/${event.target.dataset.id}`
fetch(companyUl, configObj)
let companySection = document.getElementById(`${event.target.dataset.id}`)
companySection.remove()
})
)
}
getRvs(event){
let companyId = event.target.dataset.id
this.rvsAdapter.fetchRvs(event)
.then(rvData => this.filterCorrectRvs(rvData,companyId))
// .then(rvData => this.rvsData(rvData))
// .then(rvData => this.rvsData(rvData))
}
filterCorrectRvs(rvData, companyId){
this.itemsSelected = []
this.compIdInInteger = parseInt(companyId, 10)
this.rvsArray = rvData.data.rvs
for(let item of this.rvsArray){
if(item.company_id === companyId){
this.itemsSelected.push(obj)
this.rv.renderRVsToDom(this.itemsSelected)
} else {
return this.itemsSelected
}
}
}
}
```

<h3>company: </h3>
This is where most of the html with datas that already being populated by the Companies class will be rendered when requested by the user:


```
class Company {
constructor(id,name, address, city, state, zipcode, phonenumber, building_number,email, rvs){
this.id = id,
this.name = name,
this.address = address,
this.city = city,
this.state = state,
this.zipcode = zipcode,
this.phonenumber = phonenumber,
this.building_number = building_number,
this.email = email,
this.rvs = rvs
}
createCompanyCard(){
let companyList = document.querySelector(“div.company-card-container”)
let card = document.createElement(‘section’);
card.setAttribute(“id”,`id-${this.id}`)
card.className = “company-card”;
let companyInfo = document.createElement(‘div’);
companyInfo.className = “company-info”;
// debugger
let companyName = document.createElement(“h3”);
companyName.innerText = this.name;
let coZipCode = document.createElement(“ul”)
coZipCode.innerText = `zip code: ${this.zipcode}`
companyInfo.appendChild(companyName);
let rvListDiv = document.createElement(“div”)
rvListDiv.setAttribute(“id”,`${this.id}`)
// CREATE RV
let coDetailsBtn = document.createElement(“button”)
coDetailsBtn.setAttribute(“class”,”btn btn-dark”)
coDetailsBtn.setAttribute(“id”,`company-rvs`)
coDetailsBtn.setAttribute(“data-id”,`${this.id}`)
coDetailsBtn.innerText = “Add Rvs”
// SHOW LIST OF RVs
let rvList = document.createElement(“button”)
rvList.setAttribute(“class”,”btn btn-info”)
rvList.setAttribute(“id”,`show-rvs`)
rvList.setAttribute(“data-id”,`${this.id}`)
rvList.innerText = “Show Rvs”
// RV Uls
let rvUls = document.createElement(“div”)
rvUls.setAttribute(“class”,`rv-form-${this.id}`)
// EDIT Company Button
let coEditBtn = document.createElement(“button”)
coEditBtn.setAttribute(“class”,”btn btn-warning”)
coEditBtn.setAttribute(“id”,”update-company”)
coEditBtn.setAttribute(“data-id”, `${this.id}`)
coEditBtn.innerText = “Edit”
// DELETE Company Button
let coDeleteBtn = document.createElement(“button”)
coDeleteBtn.setAttribute(“class”,”btn btn-danger”)
coDeleteBtn.setAttribute(“id”,”delete-company”)
coDeleteBtn.setAttribute(“data-id”, `${this.id}`)
coDeleteBtn.innerText = “Delete”
let companyAddress = document.createElement(“li”);
companyAddress.innerText = `street: ${this.address}`
companyInfo.appendChild(companyAddress);
let companyCity = document.createElement(“li”);
companyCity.innerText = `city: ${this.city}`;
companyInfo.appendChild(companyCity);
let ul = document.createElement(‘ul’)
// ul.innerHTML = “<b>List of Rvs:</b>”
companyInfo.appendChild(ul)
card.appendChild(coZipCode);
card.appendChild(companyInfo)
card.appendChild(rvListDiv)
ul.appendChild(coDetailsBtn);
ul.appendChild(coDeleteBtn)
ul.appendChild(coEditBtn)
ul.appendChild(rvList)
card.appendChild(rvUls)
companyList.appendChild(card)
}
}

```

<h3>Rvs Class: </h3>
This class is building on creating a logic for the company’s nested resource, Rv. Many logics and html rendering tags are coded here.

```
class Rv{
constructor(name, capacity, rate_per_day, company_id){
// this.id = id,
this.name = name,
this.capacity = capacity,
this.rate_per_day = rate_per_day,
this.company_id = company_id
this.rvAdapter = new RvsAdapter()
this.companyAdapter = new CompaniesAdapter()
this.rvFormDiv = document.getElementsByClassName(“rv-form”)
}
renderRvForm(event){
let form = document.getElementsByClassName(`rv-form-${event.target.dataset.id}`)[0]
let rvForm = document.createElement(“form”)
let rvFormDiv = document.createElement(“div”)
rvFormDiv.setAttribute(“class”,”form-group”)
// RV NAME
let rvNameLabel = document.createElement(“label”)
rvNameLabel.setAttribute(“for”,”rv-name”)
rvNameLabel.innerText = “Rv’s Name: “
let rvNameInput = document.createElement(“input”)
rvNameInput.setAttribute(“class”,”form-control”)
rvNameInput.setAttribute(“id”, `${event.target.dataset.id}`)
rvNameInput.setAttribute(“placeholder”, “Enter an RV name”)
// RV CAPACITY
let rvCapacityLabel = document.createElement(“label”)
rvCapacityLabel.setAttribute(“for”,”rv-capacity”)
rvCapacityLabel.innerText = “Maximum Capacity: “
let rvCapacityInput = document.createElement(“input”)
rvCapacityInput.setAttribute(“class”,”form-control”)
rvCapacityInput.setAttribute(“id”, `${event.target.dataset.id}`)
rvCapacityInput.setAttribute(“placeholder”, “Enter number of guests this vehicle can fit”)
// RV Price-per-dat
let rvDailyPriceLabel = document.createElement(“label”)
rvDailyPriceLabel.setAttribute(“for”,”rv-daily-price”)
rvDailyPriceLabel.innerText = “Price/day: “
let rvDailyPriceInput = document.createElement(“input”)
rvDailyPriceInput.setAttribute(“class”,”form-control”)
rvDailyPriceInput.setAttribute(“id”, `${event.target.dataset.id}`)
rvDailyPriceInput.setAttribute(“placeholder”, “Enter daily rate”)
// SUBMIT
let rvSubmitBtn = document.createElement(“button”)
rvSubmitBtn.innerText = “Submit”
rvSubmitBtn.setAttribute(“data-id”,`${event.target.dataset.id}`)
// rvSubmitBtn.setAttribute(“id”,`${event.target.id}`)
rvSubmitBtn.setAttribute(“type”,”submit”)
rvSubmitBtn.setAttribute(“value”,”Add Rv”)
// CANCEL
let rvCancelBtn = document.createElement(“button”)
rvCancelBtn.innerText = “Cancel”
rvCancelBtn.setAttribute(“data-id”,`${event.target.dataset.id}`)
// rvSubmitBtn.setAttribute(“id”,`${event.target.id}`)
rvCancelBtn.setAttribute(“type”,”cancel”)
rvCancelBtn.setAttribute(“value”,”Cancel Form”)
rvForm.appendChild(rvNameLabel)
rvForm.appendChild(rvNameInput)
rvForm.appendChild(rvCapacityLabel)
rvForm.appendChild(rvCapacityInput)
rvForm.appendChild(rvDailyPriceLabel)
rvForm.appendChild(rvDailyPriceInput)
rvForm.appendChild(rvSubmitBtn)
rvForm.appendChild(rvCancelBtn)
rvFormDiv.appendChild(rvForm)
form.appendChild(rvFormDiv)
rvSubmitBtn.addEventListener(“click”, (e)=>{
e.preventDefault();
this.postRv(e)
})
}
postRv(event){
const form = event.target.parentElement
let eventId = event.target.dataset.id
const configurationObject = {
method: “POST”,
headers: {
“Content-Type”: “application/json”,
“Accept”: “application/json”
},
body: JSON.stringify({
“name”: form[0].value,
“capacity”: form[1].value,
“rate_per_day”: form[2].value,
“company_id”: eventId
})
};
this.rvAdapter.postRvToApi(eventId,configurationObject)
}
renderRVsToDom(item){
let compInfo = document.querySelector(“div.company-info”)
let rvsDiv= document.createElement(‘div’)
rvsDiv.setAttribute(“class” , “div-of-rvs”)
let rvLi = document.createElement(“li”)
rvLi.setAttribute(“id”,`${item.id}`)
let rvDeleteAnchor = document.createElement(“button”)
rvDeleteAnchor.setAttribute(“id”,`${item.id}`)
rvDeleteAnchor.innerText = “delete”
compInfo.appendChild(rvsDiv)
rvsDiv.appendChild(rvLi)
rvLi.innerText = `name: ${item.name} — capacity: ${item.capacity} — price: ${item.rate_per_day}`
rvLi.appendChild(rvDeleteAnchor)
}
```

<h3>Adapters: </h3>
Adapters are used solely as an interface, a dividing line to take off and landing datas that are CRUD to and from database(my Rails api). For instance, my CompanyAdapter might looks like this:
```
Class CompaniesAdapter{
constructor(){
this.baseUrl = “http://localhost:3000/api/v1/companies"
}
async getCompanies() {
try {
const response = await axios.get(this.baseUrl);
return response
} catch {
alert(“Cannot load companies into database”)
}
}
async postCompanyToApi(configObject){
return fetch(this.baseUrl, configObject)
.then(res => res.json())
.catch(error => alert(“Cannot save this company record into database” + error))
}
}
```

and my RvAdapter looks like this:

```
Class RvsAdapter {
constructor(){
this.baseUrl = `http://localhost:3000/api/v1/companies`
}
postRvToApi(id,configObject){
let rvsUrl = this.baseUrl + `/${id}/rvs`
return fetch(rvsUrl, configObject)
.then(res => res.json())
.catch(error => alert(“Cannot save this RV record into its associated company” + error))
}
fetchRvs(event) {
// OLDER VERSION
// let rvsUrl = this.baseUrl + `/${event.target.dataset.id}/rvs`
let rvsUrl = this.baseUrl + `/${event}/rvs`
try {
let response = axios.get(rvsUrl);
return response
} catch {
alert(“Cannot load Rvs from database”)
}
}
}
```

<h2>Stretch Goals:</h2>

My hope for the next few weeks and beyond is to build more features and more interactions between my front end and back end. Some of the are:
building a filtering feature that when a “show Rvs” button clicks, a user can see the entire Rvs that belong to a specific company’s record.
a user can delete Rvs of specific record
Building an authentication features such as JWT or OAuth
Building user and reservation models so a user can rent an RV and travel the world, except to the moon and back :D
In conclusion, this app is a living breathing app and it may change. Please follow my frontend and backend repositories on GitHub to learn more about, and possibly contribute to it to make it much better, together!


***Thank you***










