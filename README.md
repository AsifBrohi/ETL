<a name="readme-top"></a>

# **Group Project - Café ETL Pipeline**
<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#project-background">Project Background</a>
      <ul>
        <li><a href="#design-choices">Design Choices</a></li>
        <li>
        <a href="#proof-of-concept">Proof of Concept</a>
        <li><a href="#moving-etl-to-cloud">Moving ETL to Cloud</a>
        </li>
        <li><a href="#automating-deployment-and-visualising-data">Automating Deployment and Visualising Data</a>
        </li>
      </ul>
    </li>
    <li><a href="#getting-started">Getting Started</a></li>
    <li><a href="#usage">Usage</a></li>
    <li>
      <a href="#development-roadmap">Development Roadmap</a>
      <ul>
        <li><a href="#week-one-%2D-setup-proof-of-concept-etl-pipeline">Week One - Setup proof of concept ETL pipeline</a></li>
        <li><a href="#week-two-%2D-move-etl-pipeline-to-the-cloud">Week Two - Move ETL pipeline to the cloud</a></li>
        <li><a href="#week-three-%2D-automate-deployment-and-visualise-data">Week Three - Automate Deployment and Visualise Data</a></li>
        <li><a href="#future-plans">Future Plans</a></li>
        </ul>
      </li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>

<!-- PROJECT BACKGROUND -->
## **Project Background**
We were approached by a client who had seen unprecedented growth in their café, expanding from a single branch to over a hundred outlets around the country. Currently daily transaction data is stored to a CSV throughout the day, at 8pm this is processed and stored to a local database, with no convenient method of collating and querying data from all outlets. This made it very difficult to gather meaningful, company-wide data. 

As a business, they wanted to identify ways of attracting customers and identifying sales trends across all outlets in order to capitalise on untapped revenue streams; however this was not possible with their existing technical setup.

Following discussion with the client, we agreed upon the following:
-  **To develop an automated, fully scalable ETL (Extract, Transform, Load) pipeline** to handle the high volumes of data generated by the business and bring the data from the 100+ branches together into a central repository via use of a data warehouse
- **Make use of application monitoring software to produce operational metrics** such as how often the system is run, errors, up-time and more - this is useful for us as developers as it allows us to ensure our solution is functioning successfully and aid us in identifying bugs
- **Connect the data warehouse to analytics software** in order to create business intelligence analytics for the client. Some questions they would like answered include:
  - What is the best selling product across all stores in a given month?
  - What product is the most popular across all stores?
  - Which store is the most/least profitable?
  - When is the ‘rush hour’ of the business?
  - Which payment type is more popular by store?
  - Does price influence popularity (if at all?)
  - Are people mostly buying one item or multiple in a single transaction? 

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### **Design Choices**
 - Insert information that needs to go here
<p align="right">(<a href="#readme-top">back to top</a>)</p>

### **Proof of Concept**
### **Data Normalisation and Schema**
We first needed took a look at a CSV from the company to decide on what tables we needed. The data was normalised to Third Normal Form and checked with our project owner/technical lead. Through the normalisation process, we also identified relationships that would need to exist in order for the data to be queryable.

**Resulting tables:**
##### **Note:** asterisk(*) = primary key; *italics* = foreign key


- **Transactions** 
  | transaction_id* | timestamp | *store_id* | total_price | *payment_method_id*
  | ----------- | ----------- |----------- | ----------- |----------- |
  | 1 | 25/08/2021 09:00:00 | 1 |  2.45| 1 |
  | 1 | 25/08/2021 09:02:00 | 1 |  7.95| 1 |
- **Payment_method**
  | payment_method_id* | payment_method |
  | ----------- | ----------- |
  | 1 | CARD |
  | 2 | CASH |
- **Stores**
  | store_id* | store_name |
  | ----------- | ----------- |
  | 1 | Chesterfield |
  | 2 | Uppingham |
- **Products**
  | product_id* | product_name | price |
  | ----------- | ----------- | ----------- |
  | 1 | Regular Latte | 2.45|
  | 2 | Large Flavoured Latte - Hazelnut | 2.75|
- **Sales** 
  | sales_id* | *transaction_id* | *product_id* |
  | ----------- | ----------- | ----------- |
  | 1 | 1 | 1|
  | 2 | 2 | 1 |
  | 2 | 2 | 1 |

We discussed breaking down the products table further, into separate size, type and flavour columns, however after considering the impact such a breakdown could have on join times which could impact querying the data, we ultimately decided to stick with a single table for products.

For a full breakdown of the normalisation process, please see <a href="https://github.com/DELON8/group-5-data-engineering-final-project/blob/main/supplementary_documentation/data_normalisation.pdf">here for documentation</a>.

With the data normalised, we were then able to design our schema. 
[final-schema](https://user-images.githubusercontent.com/116800613/213325530-d6d0646a-058b-4f1d-9206-85cd0263bfa6.png)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### **Setting up docker for PostgreSQL and Adminer**
Let’s create a directory, postgres, and then create a docker-compose.yml file in that directory:

```bash
mkdir postgres
cd postgres
touch docker-compose.yml
```

Basically, here, we will specify the services we are going to use and set up the environment variables related to those.

We will change this file multiple times throughout this article.

Add the following in the docker-compose.yml file we just created:

```yaml
version: "3.1"
services:
  db:
    image: postgres
    container_name: demo
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: postgres
      
    ports:
      - 5432:5432
    volumes:
      -  ./demo_db:/var/lib/postgresql/data
  adminer:
    image: adminer
    container_name: adminer_container_demo
    restart: always
    ports:
      - 8080:8080
volumes:
  demo_db:
  ```

We specified the name of our PostgreSQL container as demo and the Docker image to be used is postgres.

The next thing we need to specify is the environment variables, i.e. the user, password, and database. If you don’t specify the user, by default it will be <b>root</b>.

Volume is mounted at /var/lib/postgresql/data. Inside the container, this directory is where Postgres stores all the relevant tables and databases.

Now, after creating the .yml file, we need to run the following command in the same directory where the .yml file is located:

```bash
docker-compose up
```

This will pull the Docker image (if the image is not available locally, it will pull from Docker Hub) and then run the container.

We can check the status with:

```bash
docker-compose ps
```
![Screenshot 2022-12-07 at 19 31 47](https://user-images.githubusercontent.com/113560228/206277976-688302ec-2713-435a-9d34-8f911ed71819.png)

This will show the name of the container, command, and state of the container, which shows, for example, that the container is running. It also shows port mapping.

### **Connect to the PostgreSQL Database Running in a Container**

Now, we can go to our browser and go to localhost:8080 for Adminer. As Adminer runs on the same Docker network as PostgreSQL, it can access the Postgres container via port 5432 (or simply, by the container’s name).

![Screenshot 2022-12-07 at 19 41 37](https://user-images.githubusercontent.com/113560228/206280979-f1cfb886-e6a9-458e-abff-f5fd53bb4f9c.png)


### **Data Extraction**
![DataInChecks drawio](https://user-images.githubusercontent.com/116560975/207384408-c7846e88-62be-4846-9258-e5805449943e.png)

Although the CSV file we used for our POC had no headers, we thought about other cases that we could encounter that could potentially crash our app. For example:

- We may receive a file with missing columns.
- We may receive a file with or without headers. 
- We may receive a file with incorrect headers.

After careful consideration, we came up with the following:
- If the number of columns the extraction fails and the script outputs an error. 
- If the number of columns is right but there are no headers. The script outputs a warning, adds the headers and extracts the data and makes it available as a pandas dataframe.
- If the headers are present and correct, just extract the data and present it as a pandas dataframe.


Furthermore, seven unit tests have been developed to help minimize regression.

![image](https://user-images.githubusercontent.com/116560975/207386669-ed25ddb8-a9fe-4392-bb75-8558d8e84a56.png)

### **Move ETL to Cloud**

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### **Automate Deployment and Visualise Data**

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## **Getting started**
- Instruct the user on how to get their own version of our app set up, direct them to files they need to run, include screenshots, etc.

## **Usage**
- Describe what insights we can gain
- Demo?
- Demonstrate evidence of testing

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## **Development Roadmap**
### Week One - Setup basic ETL pipeline (Proof of concept)
- [ ] Agree on ways of working and definition of done
- [ ] Create script to generate database
- [ ] Normalise data and create schema
- [ ] Set up docker for PostgreSQL and Adminer
- [ ] Extract, transform and load data to database

### Week Two - Move ETL pipeline to the cloud
- [ ] Set up database in AWS Redshift
- [ ] Set up AWS Lambda to be triggered by and S3 event
- [ ] Modify Labda to run ETL functions
- [ ] Set up cloudformation


### Week Three - Automate Deployment and Visualise Data
- [ ] Set up Grafana
- [ ] Set up Infrastructure Monitoring
- [ ] Visualise Sales Data
- [ ] Set Up CI/CD Pipeline

### Future Plans
- [ ] Break up Lambda function 
- [ ] Separate the extract Lambda in the public subnet from the transform and load Lambda in the private subnet



<p align="right">(<a href="#readme-top">back to top</a>)</p>

## **Contributing**

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## **Contact**
- Links to our respective LinkedIns

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## **Acknowledgments**

<p align="right">(<a href="#readme-top">back to top</a>)</p>
