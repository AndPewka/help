
# Antifraud #
## Opencv ##
```ruby
apt-get install -y cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
apt-get install -y python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev

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

### Drop expiration proxy ###
```ruby
Proxy.where(state: [:working, :disabled]).select { |proxy| proxy.expiration < Time.current }.each { |proxy| proxy.update! state: :archived }
Proxy.where(state: :archived).each {|proxy| ProxyRemoveWorker.new.perform(proxy.id, proxy.proxy_group_id)}
```
### Add proxy ###
```ruby
active_proxy_groups_ids = [2, 23, 27] # ru ua eu
active_proxy_groups_ids.each do |id|
  resp = Faraday.get "https://api.antifraudsms.com/api/v1/share_resources/get_proxies_by_group_id", { id: id } , { "Authorization": ENV["USER_API_TOKEN"] };proxies = JSON.parse(resp.body);proxies.each { |p| p["ip"] = p.delete("host"); Proxy.create! p }
end
```

### Mail reader ###
```ruby
resp = Faraday.get "https://api.antifraudsms.com/api/v1/share_resources/get_mail_accounts", nil , { "Authorization": ENV["USER_API_TOKEN"] };accs = JSON.parse(resp.body);accs.each { |a| a["setting_id"]=1;a.delete("id"); Account.create! a }
```


### Accounts ###
```ruby
Account.select("DISTINCT ON (login, service_id) *").all.each do |account|
  if Account.where(login: account.login, service_id: account.service_id).count > 1
    account.destroy
  end
end
```

## DATABASE ##

### Get dump ##
```ruby
docker exec backend_postgres_1 pg_dump -U postgres antifraud_development > backup.dump
```
### Put dump ###
```ruby
docker container prune
rails db:create
docker cp backup.dump backend_postgres_1:/backup.dump
docker exec -it backend_postgres_1 bash
psql -U postgres -d antifraud_development < backup.dump
```
