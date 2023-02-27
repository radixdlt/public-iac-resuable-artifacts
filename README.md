# public-iac-resuable-artifacts

WARNING: These actions are not useful for developers outside the radixdlt, radixworks, metaverse or metapass organisation as the workflows and steps are highly specialized for usage in the company. Since some repositories are public, they are not able to access private or internal repositories.

Reusable workflows:
.github/workflows/

## Docker build and push

There is a variety of settings and steps that are done to perform a docker push and this reusable step aims to both simplify the process for developers aswell as improve the process. Developers profit from reduced effort to write github actions, additional features often not used yet like image scanning and linting and also improve their build times with mandatory caching.
Devops and Security profit from a single point of maintenance in case of required updates for github actions.

