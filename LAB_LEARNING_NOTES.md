# Lab 5 Docker Learning Notes

A summary of questions from the lab and TA Q&A for study and review.

---

## 1. Docker Commands

### Q: What do `-t` and `-f` mean?

- **`-t`**: `--tag`, assign a name to the image (e.g., `-t mlip-training`)
- **`-f`**: `--file`, specify the Dockerfile path (e.g., `-f docker/training/Dockerfile`)

### Q: What does `docker run --rm -v wine_model_storage:/app/models mlip-training` mean?

- `--rm`: Automatically remove the container when it exits
- `-v wine_model_storage:/app/models`: Mount a named volume to store the model
- `mlip-training`: The image to use

### Q: What does `-p` mean?

- **`-p`**: Port mapping, format `-p <host_port>:<container_port>`
- Example: `-p 8081:8080` → Host port 8081 maps to container port 8080

### Q: Is `--rm` "run and remove"?

Yes. **Run** the container and **remove** it automatically when it exits.

### Q: How do I decide which ports to use for port mapping?

- **Container port**: Defined by the application (e.g., Flask's `app.run(port=8080)`)
- **Host port**: Your choice—any available port (e.g., 8081, 3000)

### Q: What is `docker compose up --build`?

- `docker compose up`: Start all services defined in docker-compose.yml
- `--build`: Build images before starting (rebuilds even if they already exist)

### Q: Does Docker Compose automatically read docker-compose.yml?

Yes. When you run `docker compose up` from the project root, it reads `docker-compose.yml` in that directory.

---

## 2. Dockerfile

### Q: What does `CMD ["python", "train.py"]` mean?

- `CMD`: The default command to run when the container starts
- `["python", "train.py"]`: Exec form—runs `python train.py`

### Q: What is a Dockerfile and how does it help containerize?

- A file that defines how to build an image
- Steps from base image → install dependencies → copy code → CMD to package the service into a runnable image

---

## 3. Volumes and Data Storage

### Q: Do I need to mount both model volume and logs for inference?

- **Model volume**: Required—inference needs to load the model
- **Logs**: Optional—only if you want to view logs on the host

### Q: Docker run fails with `-v $(pwd)/logs` when path has spaces?

- Add quotes: `-v "$(pwd)/logs:/app/logs"`

---

## 4. API and curl

### Q: What do the parts of the curl command mean?

- `-X POST`: HTTP method
- `-H 'Content-Type: application/json'`: Tells the server the body is JSON
- `-d '{"input": [...]}'`: Request body

### Q: Where do Content-Type and data come from?

- **Content-Type**: HTTP standard
- **Data**: Example—typically a sample row from the sklearn Wine dataset

---

## 5. TA Questions and Answers

### 1. Why is the log saved in the folder?

**Your answer**: Because it's specified in the yml file in the volume section.

**Clarification**: Logs are saved via **bind mount** `./logs:/app/logs` to the host's `./logs` folder. When the inference service writes to `/app/logs/predictions.log`, it appears directly on the host. The named volume is for the model; logs use a bind mount.

### 2. Are the logs on your local machine?

**Your answer**: TA said `docker volume inspect wine_model_storage` helps find the file on the local machine, so yes.

**Clarification**:
- **Logs**: Stored in `./logs` in the project directory—directly visible on the host
- **Model (.pkl)**: Stored in named volume `wine_model_storage`—use `docker volume inspect wine_model_storage` to see the Docker-managed path

### 3. Where are the pkl files?

**Your answer**: In volume storage (wine_model_storage).

**Correct**: The model is in named volume `wine_model_storage`, mounted to `/app/models` in the container.

### 4. Explain again what's the feature of Docker saving the log.

**You can say**: Docker uses a **bind mount** (`./logs:/app/logs`) to map the host's `./logs` to the container's `/app/logs`. When the container writes to `/app/logs/predictions.log`, it appears immediately on the host's `./logs`—you can view or edit logs on the host without entering the container.

---

## 6. Named Volume vs Bind Mount

| Type | Purpose | Syntax | Storage Location |
|------|----------|--------|-------------------|
| **Named Volume** | Model, shared between containers | `wine_model_storage:/app/models` | Docker-managed (see `docker volume inspect`) |
| **Bind Mount** | Logs, direct host access | `./logs:/app/logs` | Host path you specify |

---

## 7. Other

### Q: What are the 3 problems in server.py?

- Import warnings (flask, numpy, joblib) because these packages are installed in Docker, not locally
- The code is fine—it runs correctly inside Docker
- To fix warnings: run `pip install -r docker/inference/requirements.txt`

### Q: Correct git set-url command?

```bash
git remote set-url origin "https://github.com/joannachang1028/mlip-docker-lab-s26.git"
```

---

## 8. Lab Flow Summary

1. `docker run --rm -v wine_model_storage:/app/models mlip-training` — train
2. `docker run --rm -p 8081:8080 -v wine_model_storage:/app/models -v "$(pwd)/logs:/app/logs" mlip-inference` — inference
3. `docker compose up --build` — start all services
4. `docker compose down -v` — remove volume to verify lifecycle

---

*Last updated: 2026-02*
