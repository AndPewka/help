
# Antifraud #
## Opencv ##
```ruby
apt-get install -y cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
apt-get install -y python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev
# sudo apt-get install -y python-dev-is-python3 libtbbmalloc2 libjpeg-dev libpng-dev libtiff-dev libdc1394-dev libopencv-dev libavcodec-dev libavformat-dev libavutil-dev libswscale-dev python3-numpy

git clone -b 2.4 --single-branch --progress https://github.com/opencv/opencv.git /tmp/opencv

mkdir /tmp/opencv/release && \
cd /tmp/opencv/release && \
cmake -D CMAKE_INSTALL_PREFIX=/usr/local \
-D CMAKE_BUILD_TYPE=Release \
-D INSTALL_PYTHON_EXAMPLES=OFF \
-D INSTALL_C_EXAMPLES=OFF \
-D BUILD_TESTS=OFF \
-D BUILD_PERF_TESTS=OFF \
-D BUILD_EXAMPLES=OFF \
-D WITH_QT=OFF \
-D WITH_OPENGL=OFF \
-D WITH_FFMPEG=OFF \
-D FORCE_VTK=OFF \
-D ENABLE_PRECOMPILED_HEADERS=OFF .. && \
make -j$(nproc) && sudo make install

gem install ruby-opencv -- --with-opencv-dir=/usr/local
```

## Fetch data ##

### Add proxy ###
```ruby
active_proxy_groups_ids = [2, 23, 27] # ru ua eu
active_proxy_groups_ids.each do |id|
  resp = Faraday.get "https://api.antifraudsms.com/api/v1/share_resources/get_proxies_by_group_id", { id: id } , { "Authorization": ENV["USER_API_TOKEN"] };proxies = JSON.parse(resp.body);proxies.each { |p| p["ip"] = p.delete("host"); Proxy.create! p }
end
```

### Drop accounts duplicate ###
```ruby
Account.select("DISTINCT ON (login, service_id) *").all.each do |account|
  if Account.where(login: account.login, service_id: account.service_id).count > 1
    account.destroy
  end
end
```

### Get mail_reader acc csv(outlook example) ###
```ruby
csv_text = File.read('accounts.csv')
csv = CSV.parse(csv_text, headers: true)
csv.take(500).each do |row|
Account.create(
    service_id: 33,
    login: row['Login'],
    password: row['Password'],
    parameters: JSON.parse(row['Parameters'].gsub(/=>/, ":")),
    setting_id: 1
)
  end
```



## DATABASE ##

### Get dump ##
```ruby
docker exec backend_postgres-1 pg_dump -U postgres antifraud_development > backup.dump
```
### Put dump ###
```ruby
docker container prune
rails db:create
docker cp dumps/postgres.dump backend_postgres_1:/postgres.dump
docker exec -it backend_postgres_1 bash
pg_restore -h localhost -U postgres -d antifraud_development -1 postgres.dump
# psql -U postgres -d antifraud_development < backup.dump
```
