Here, I am trying to build `guestbook` application. It's source code, dockerfiles and .gitlab-ci.yml are available in `guestbook` folder in this repo. 

You can try this in your GitLab enviroonment by following below steps:
- Create `guestbook` repo in GitLab. 
- Add `guestbook` folder contents in above repo.
- Set pipeline variables:
   - `CI_REGISTRY_FRONTEND` (path of registry for frontend container)
   - `CI_REGISTRY_SLAVE` (path of registry for redis slave)
   - `REG_PASSWORD` (registry password)
   - `REG_USER` (registry user)
   - `RELEASE` (release version)
- Make sure that GitLab runner has appropriate tag.
- Run pipeline, it will push images to registry. I have pushed my images at docker hub (https://cloud.docker.com/repository/list). 