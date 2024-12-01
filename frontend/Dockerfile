# Use an official Node runtime as a parent image
FROM node:14-alpine AS build

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json
COPY package.json .

# Install app dependencies
RUN npm install

# Copy the current directory contents into the container
COPY . .

# Build the app
RUN npm run build


# Serve static content on nginx
FROM nginx:alpine

COPY --from=build /app/build /usr/share/nginx/html
COPY default.conf /etc/nginx/conf.d/default.conf

# Make port 3000 available to the world outside this container
EXPOSE 3000

# Define environment variable




# Run the app when the container launches
CMD ["nginx", "-g", "daemon off;"]
