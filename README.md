# Taas

## Terraform as a (RESTful CRUD) Service

Taas provides a RESTful CRUD API for terraform. The goal is to provide a self service API that allows developers to provide the configuration for the desired infrastructure, provision it on-demand in a predicatable and consistent way, and interact with the lifecycle of their infrastructure. Behind the scenes, the service satisfiying the API requests offers not only state changes, but as well more rich features such as auto-destroy, resource pooling, authentication, and auditing.

### Installation

With a [correctly configured](https://golang.org/doc/install#testing) Go toolchain:

```sh
go get -u github.com/kmacoskey/taas
```

### Building

```sh
make
```

### Basic Use

Basic use of the API begins with a user suppliying valid terraform configuration in order to request provisioning of supplied infrastructure. A user may list the state and other useful information of infastructure the API is keeping track of. Then a user may request de-provisioning of any of the managed infrastructure.

#### Create Infrastructure

Request provisioning of new infrastructre.

```sh


new_infra=$(cat <<EOF
{
    "terraform_config": "
      provider \"google\" {
        project     = \"my-gce-project-id\"
        region      = \"us-central1\"
      }

      resource \"google_compute_instance\" \"test\" {
        name         = \"test\"
        machine_type = \"n1-standard-1\"
        zone         = \"us-central1-a\"

        network_interface {
          network = \"default\"
        }

        boot_disk {
          initialize_params {
            image = \"debian-cloud/debian-8\"
          }
        }
      "

}
EOF
)

response=$(curl --header "Content-Type: application/json" \
                --request POST \
                --data "${new_infra}" \
                http://localhost:8080/infra)
```

The API call to create infrastructure will return a Unique ID to refernence for subsequent  interaction with the lifecycle of the infrastructure.

```
infra_uuid=$(echo ${response} | jq .uuid)
```

#### List Infrastructure

Gather information about all infrastructure:

```sh
curl --header "Content-Type: application/json" \
     --request GET \
     http://localhost:8080/infras
```

#### Destroy Infrastructure

Using the Unique ID returned when creating infrastructure, a user may call for immediate deletion of the infrastructure:

```sh
curl --header "Content-Type: application/json" \
     --request DELETE \
     http://localhost:8080/infra/${infra_uuid}
```

### Testing

Run the ginkgo/gomega tests:

```sh
make test
```

