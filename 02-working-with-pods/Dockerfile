FROM localhost/a-base-image:1.0

USER root
RUN mkdir /my-special-folder 
COPY Dockerfile /my-special-folder
RUN  echo "" >> /my-special-folder/Dockerfile && \
	echo "# This is the end" >> /my-special-folder/Dockerfile && \
	echo "# Hold your breath and count to ten" >> /my-special-folder/Dockerfile && \
	echo "# Feel the earth move and then" >> /my-special-folder/Dockerfile && \
	echo "# Hear my heart burst again" >> /my-special-folder/Dockerfile && \
	echo "# For this is the end" >> /my-special-folder/Dockerfile && \
	echo "" >> /my-special-folder/Dockerfile
