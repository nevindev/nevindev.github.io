---
title: "My Resumé"
---

<!-- #### Reston, Virginia john@nevin.dev (978) 973-6262 #### -->

<name> John Nevin </name>

#### Reston, VA ####
#### john@nevin.dev ####
#### (978) 973-6262 ####

### Education ###
<subtitle>
    <div>
        <key> James Madison University </key>
        <startValue> Harrisonburg, VA </startValue>
    </div>
    <div>
        <italicValue> BS Engineering </italicValue>
        <endValue> Aug 2014 - May 2018 </endValue>
    </div>
</subtitle>

---

### Work Experience ###
<subtitle>
    <div>
        <key> EnterpriseDB (EDB) </key>
        <startValue> Bedford, MA (Remote)</startValue>
    </div>
    <div>
        <italicValue> Senior Software Engineer </italicValue>
        <endValue> Apr 2023 - Feb 2026 </endValue>
    </div>
</subtitle>

* Member of the [Trusted Postgres Architect](https://github.com/EnterpriseDB/tpa) (TPA) product team.

* Led development of an automatic file checksum validation feature to pinpoint troubleshooting of deployed clusters by comparing Trusted Postgres Architect (TPA) installations in customer environments against the released package.

* Led migration of Github Actions CI/CD workflows as part of the open-sourcing effort for TPA, increasing visibility and adoption.

* Developed a semantic similarity search tool to improve integration testing by comparing vector embeddings of customer's deployed clusters against our test clusters.

* Implemented ability for TPA to deploy a novel and widely-adopted distributed Postgres cluster architectural pattern for active-active data replication and failover management.
    
* Led development of a TPA feature for upgrading software installed on deployed clusters with minimal to no downtime in availability, backup or failover management.

<subtitle>
    <div>
        <key> CGI Federal </key>
        <startValue> Fairfax, VA </startValue>
    </div>
    <div>
        <italicValue> Senior Consultant </italicValue>
        <endValue> Jul 2018 - Apr 2023 </endValue>
    </div>
</subtitle>

* Developed Python applications to interface with third-party REST APIs to retrieve millions of events daily from many different sources and tools for forwarding, indexing and searching in Splunk for integration with Kibana and Elasticsearch.

* Incorporated robust logging into Python applications and wrote statistical searches for monitoring performance and data quality.

* Normalized, filtered and enriched terabytes of heterogeneous data with fast, consistent and maintainable searches in Splunk.

* Worked to maintain 80% coverage of all data retrieved from disparate sources against the expected state and reduced downtime for reporting data downstream.

* Fully automated the deployment of Splunk app releases with a Python program; handles backing up, configuration and deployment with a user-friendly interactive CLI wizard.

---

### Technical Skills ###
<subtitle>
    <div>
        <key>Programming Languages: </key>
    </div>
    <div>
        <endValue> Python, Elixir/Erlang, TypeScript/ES6, Java/Kotlin, Rust, SQL, HCL </endValue>
    </div>
</subtitle>
</br>
<subtitle>
    <div>
        <key>Frameworks: </key>
    </div>
    <div>
        <endValue> Phoenix, Svelte, React, Flask, OpenCV, Detectron2, pgrx </endValue>
    </div>
</subtitle>
</br>
<subtitle>
    <div>
        <key>Technologies: </key>
    </div>
    <div>
        <endValue>
        Ansible, AWS EC2, Lambda, S3, ECR, Docker, GitHub Actions, Grafana, OpenTelemetry, Prometheus, RabbitMQ, Terraform/OpenTofu, GraphQL
        </endValue>
    </div>
</subtitle>

---

### Personal Projects ###
<subtitle>
    <div>
        <key>LiftGenius </key>
        <a href="https://github.com/liftgenius" target="_blank" rel="noreferrer">Project Repository</a>
        <italicStartValue> Phoenix, PostgreSQL, RabbitMQ, AWS Lambda, AWS S3, Flask, OpenCV, Detectron2, Docker Compose, OpenTofu, Headscale/Tailscale </italicStartValue>
        <ul>
            <li>
            A full stack Phoenix application to build lifting workouts track bar-paths using AI. Utilizes Elixir channels and websockets to enable real-time collaboration for program creation. Workout days are automatically scheduled on a calendar and exercise completion is tracked for analysis.
            </li>
            <li>
            Videos of lifts are uploaded, enqueued, analyzed and processed using S3, RabbitMQ, AWS Lambda and an OpenCV Flask API respectively. The job status is shown to the user in real-time and a URL with the analyzed video is provided upon completion.
            </li>
            <li>
            Infrastructure is deployed as code using Docker Compose, OpenTofu, and Tailscale for fast, secure and consistent deployment across different environments for development, testing and production.
            </li>
        </ul>
    </div>
</subtitle>
</br>
<subtitle>
    <div>
        <key>NearMiss.me </key>
        <a href="https://nearmiss.me" target="_blank" rel="noreferrer">nearmiss.me</a>
        </br>
        <a href="https://github.com/nevindev/nearmiss" target="_blank" rel="noreferrer">Project Repository</a>
        <italicStartValue>  React, PostgreSQL, GraphQL, tailwindcss, Leaflet</italicStartValue>
        <p>A Progressive Web Application for reporting near-miss incidents between pedestrians/cyclists and motorists. Automatically records location of incident and allows users to easily provide details in dynamic forms. Thousands of map features can be displayed due to GPU-accelerated vector tile rendering. Web form questionnaires are built dynamically by parsing JSON. All data is stored locally in the browser IndexedDB for full-offline capability and user-ownership of their own data and sharing decisions</p>
    </div>
</subtitle>

---

### Awards and Achievements ###

<key> Top EDB Technical Blog Post </key>
<line>
    <startValue> EnterpriseDB </startValue>
    <endValue> Feb 2025 </endValue>
</line>

 Authored ['Representing Graphs in PostgreSQL with SQL/PGQ'](https://www.enterprisedb.com/blog/representing-graphs-postgresql-sqlpgq) which became the most-viewed post of the month with over 7,500 hits. Gained traction and generated discussion on both YCombinator Hacker News and lobste.rs boards.
</br>

<key> Accepted to JMU Summer Startup Accelerator Cohort </key>
<line>
    <startValue> James Madison University </startValue>
    <endValue> July 2017 </endValue>
</line>

 Received $4,000 in funding from the university to further develop my hackathon-winning project into a
minimum viable product in an eight week accelerator program.
</br>

<key> Third Place, Bluestone Hackathon </key>
<line>
    <startValue> Bluestone Hacks </startValue>
    <endValue> May 2017 </endValue>
</line>

Won a 24-hour Hackathon for creating a smartphone application for assisting individuals with memory loss
that addressed the hackathon theme of "safety".