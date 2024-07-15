## Local Setup

1. Clone repository
2. Create a `.env` file based on the `.env.example`
3. Update submodules with `git submodule update --init --recursive`
4. Make sure Docker is running locally
5. Run `docker compose up --build`
6. The API Gateway will be accessible at `http://localhost:<PORT>`, where `<PORT>` is the port defined in the
   `docker-compose.yml` file.

If you want to update the references of the submodules, you can run the following
command: `git submodule update --remote`.

## Important Note

When Docker Compose runs the services, the database services start up more slowly than the microservices themselves.
Therefore, you need to wait until the databases are ready to accept connections before restarting the microservices.
This issue occurs in development but should not happen in a production environment.

With Kubernetes, the pods would continuously attempt to reconnect until the databases are fully operational, mitigating
this issue.

## Architecture Overview

### API Gateway

The API Gateway is the only entry point exposed to the outside world. It receives HTTP requests from clients and routes
them to the appropriate microservices. This gateway ensures security and acts as a single point of management for client
interactions.

### Microservices

- **Users Microservice**
    - **Endpoints**:
        - `Register`: Adds a new user to the database.
        - `Subscribe`: Adds a subscription to the subscriptions database.
    - **Database**: MySQL
    - **Communication**: Via NATS with other microservices and the API Gateway.

- **Scraping Microservice**
    - **Functionality**: When the project starts, the scraping microservice runs a cron job to start the scraping
      process of the teams.

- **Competitions Microservice**
    - **Functionality**: The competitions microservice stores leagues and teams; currently, it only receives teams sent
      by the scraping
      microservice. The functionality to store leagues is still pending.
    - **Database**: MySQL

- **Notifications Microservice**
    - **Functionality**: I have created the logic in the notifications microservice to send notifications to active
      subscriptions, but the
      scraping microservice does not yet have the functionality to generate notifications and communicate with the
      notifications microservice.
    - **Database**: MySQL

- **Stats Microservice**
    - **Functionality**: (Pending) This microservice will handle statistical data.
    - **Database**: MySQL

### Docker and Docker-Compose

Each microservice is containerized using Docker to ensure consistency and portability across different environments. A
project launcher uses Docker-Compose to bring up the necessary databases and microservices together, facilitating
development and deployment.

## Benefits of the Architecture

1. **Scalability**: Each microservice can be scaled independently based on its specific needs and traffic. This
   flexibility allows efficient resource utilization and better performance under load.

2. **Isolation**: By having independent MySQL databases for each microservice, the system ensures data encapsulation and
   reduces the risk of data corruption or unintended interference between services.

3. **Modularity**: The microservices architecture promotes modular development. Teams can work on different services
   simultaneously without causing disruptions to the entire system.

4. **Maintainability**: Smaller, well-defined services are easier to manage, update, and debug. It also allows for the
   adoption of new technologies in specific parts of the system without a complete overhaul.

5. **Resilience**: Faults in one microservice do not affect the entire system. This isolation improves the overall
   resilience and reliability of the application.

![Application Architecture](./docs/architecture.png)

## HTTP Requests

You can use the following requests to interact with the API endpoints:

### Users Microservice

- **Register a User**
    - **Method**: POST
    - **URL**: `http://localhost:<PORT>/api/users/register`
    - **Body**:
      ```json
      {
        "name": "name example",
        "email": "user@example.com",
        "address": "address example"
      }
      ```

- **Subscribe a User**
    - **Method**: POST
    - **URL**: `http://localhost:<PORT>/api/users/subscribe`
    - **Body**:
      ```json
      {
        "userId": "ecad02e5-cb0c-4f70-9b1b-8491740e3d54",
        "teamId": "a880d039-bb4e-48c9-99ae-6b8734195ca0",
        "leagueId": "15ed7a50-db72-4f13-afbc-af4268b492bc"
      }
      ```

## Pending work

- [ ] Add stats microservice.
- [ ] Add notifications when a live game is in progress.
- [ ] Add missing endpoints.
- [ ] Add unit tests.
- [ ] Add integration tests.
- [X] Add CI/CD.
- [X] Add Kubernetes.
- [ ] Add monitoring.
- [X] Upload the project to a cloud provider.
- [X] Add production environment.
- [ ] Add Caching with Redis.
- [ ] Add Authentication and Authorization.