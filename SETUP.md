az group create --location westeurope --resource-group rg-lgu
az mysql flexible-server create --location westeurope --resource-group rg-lgu --name leclash-db --vnet leclash-vnet --subnet leclash-db-subnet
az network vnet subnet create --resource-group rg-lgu --vnet-name leclash-vnet --name leclash-app-subnet --address-prefixes 10.0.1.0/24  --delegations Microsoft.Web/serverFarms --service-endpoints Microsoft.Web
# az webapp up --resource-group myresourcegroup --location westus2 --plan testappserviceplan --sku P2V2 --name mywebapp
az webapp vnet-integration add --resource-group rg-lgu --name leclash-rest --vnet leclash-vnet --subnet leclash-app-subnet
az webapp config appsettings set --name leclash-rest --resource-group rg-lgu --settings SPRING_PROFILES_ACTIVE=mysql,spring-data-jpa MYSQL_URL="jdbc:mysql://leclash-db.mysql.database.azure.com:3306/flexibleserverdb?useUnicode=true&serverTimezone=UTC&useSSL=true" MYSQL_USER="panickyTruffle8" MYSQL_PASS="SFESwwiEurPJOmbZcIHcWw" WEBSITE_DNS_SERVER="168.63.129.16" WEBSITE_VNET_ROUTE_ALL=1

az storage account create --name leclash --resource-group rg-lgu --kind StorageV2
az storage blob service-properties update --account-name leclash --static-website --404-document 404.html --index-document index.html
az storage blob upload-batch --source dist/ --destination \$web --account-name leclash
az storage account show --name leclash --query primaryEndpoints.web