# SpotifyAiArt

This application allows a user to generate Ai Art based on their Spotify music "Mood" when matched to another person.

## Technical Stack & Justifications
### Fronted
- Next.js
  - Reasoning: I just want to learn Next.js. There's no technical justification, the same could probably be achieved with a single HTML page being served in an s3 bucket as the front end will only require 1-3 frontend flows.


### Backend 
- Flask/Python
  - Reasoning: The API will only serve to call spotify's api for music features and to aggregate features into a qualitative mood
- Spotify API
   - Reasoning: Well documented, and many applications have served as POC's for using spotifyAPI vs Apple music/Soundcloud etc.
- Stable Diffusion
    -  Reasoning: Open source AI Model, but requires to self-host on infra which will add additional operational overhead as well as complexity.
- PostgreSQL
    - A common database will be needed to store users and art. This will introduce cost savings measures in compute time for the model (no need to recompute multiple times for the same match).
- Redis Queue (Roadmap for version 2)
  - To save on compute for the model deployment it may be possible to host a Redis queue on the backend host to queue messages for a spot instance. Also allows modularity at scale (e.g if the model fails, user can come back to image rather than failing)
- Docker
  - Reasoning: Standard containerization for application portability, and versioning.
- Docker-Compose
   - Reasoning: For starting up front end and backend services in conjuction. K8s or similar would be overkill unless this application scaled up in users dramatically.

### Hardware 
For all hardware, cost-saving measures are the priority.
- AWS EC2
- AWS Spot Inference GPU or AWS Labmda

## Security Controls
- Package Management
    - Dependabot
- SAST
  - CodeQL
  - GHAS Secret Scanning
- User Authentication
  - Users will authenticate with the spotify API.
  - User email will be collected to matches, and generated images
- User Session Management
  - Spotify user tokens must be enforced with every API call to fetch user data.
  - Spotify user tokens will be used to manage the user session.
  - Spotify user tokens must be rotated or rerequested from the user for API calls that provide user data on matches,
    or previously generate images (i.e) stale tokens should be prevented from authenticating and requesting SpotifyAIArt Data
- Rate Limiting
  - Rate Limit the api's to avoid vulnerabilities relating to brute forcing tokens
 - Api sanitization
  - All inputs for API's access the Postgresql database will need to be sanitized for sql vulnerabilities.
- Network Controls
  - Databases will be in a private subnet in it's own VPC
  - Instances or lambda functions will in a private vpc
  - Only the frontend instance will be accessible in a public subnet in vpc with an internet gateway attached.
  - Frontend instance will have port 80 exposed and 443 will reroute to 80
  - Backend instance will only accept connections on port xxxx from frontend.
  - Database will only accept connections on port xxxx from backend.
  - Model instance will only acception connections on port xxxx from backend subnet
- Infrastructure Controls
  - OS will be based on RHEL UBI
  - SELINUX must be enforced
 
## API
### Top Level Domain
` www.spotifyart.ai`

### Client URLs v1
#### Home 
`GET wwww.spotifyart.ai/`

### User Matching Art Generation
`GET wwww.spotifyart.ai/discover`

### Music Art Synthesis Service API v1
### Spotify
`GET /api/v1/music/mood `
  - retrieve's a user's spotify mood based of their top listen playlist.

`POST /api/v1/ai/art `
- retrieves's a stable diffusion art inference.
  


### Client URLS v2
` www.spotifyart.ai/match`
## Diagrams

![Infrastructure](https://gcdnb.pbrd.co/images/CLhGstsSQWCm.png?o=1)
![Databsae](https://gcdnb.pbrd.co/images/kLQ8eWRFIWBI.png?o=1)
