TraceAssist Instrumentation ActionAutomate the instrumentation of your Kubernetes applications with a single GitHub Action.The TraceAssist Action integrates with your CI/CD pipeline, calling the hosted TraceAssist service to intelligently analyze your Kubernetes manifests. It automatically injects the necessary annotations and configurations for seamless, zero-touch auto-instrumentation with the OpenTelemetry Operator.✨ How It WorksThis action simplifies observability by decoupling instrumentation from your deployment logic.Your CI/CD Workflow Runs: On a git push, your workflow builds your application's Docker image as usual.Call the TraceAssist Action: The action securely sends your deployment.yaml content to the hosted TraceAssist API. The API analyzes the manifest, injects the required OpenTelemetry annotations, and returns the modified file.Deploy Instrumented App: Your workflow then uses the modified, "observability-ready" manifest to deploy your application, which is automatically instrumented by the OpenTelemetry Operator running in your cluster.✅ PrerequisitesBefore using this action, you must have the following components set up in your target Kubernetes cluster.1. Install the OpenTelemetry OperatorThe operator is the engine for auto-instrumentation. You can install it via Helm:helm upgrade --install opentelemetry-operator open-telemetry/opentelemetry-operator \
  --namespace opentelemetry-operator-system --create-namespace
2. Configure a CollectorYou must have an OpenTelemetry Collector running in your cluster to receive telemetry data.3. Apply TraceAssist ConfigurationsYou need to apply two manifest files from this repository to your cluster. These only need to be applied once per namespace.RBAC Permissions: This creates the necessary ServiceAccount and RoleBinding. You can apply it directly from this repository:kubectl apply -f https://raw.githubusercontent.com/your-username/traceassist-action/main/templates/traceassist-rbac.yaml
(Note: You may need to edit the file to change namespace: default to your application's namespace.)Instrumentation Resource: This tells the OTel Operator how to instrument your pods.kubectl apply -f https://raw.githubusercontent.com/your-username/traceassist-action/main/templates/instrumentation.yaml
(Note: Be sure to edit this file to set the correct endpoint for your OpenTelemetry Collector.)🚀 UsageAdd a step to your existing CI/CD workflow that uses your-github-username/traceassist-action@v1. This step should come after you build your Docker image but before you deploy to Kubernetes.Example WorkflowHere is a complete example workflow. You can use this as a template for your own .github/workflows/deploy.yml file.name: Build, Instrument, and Deploy

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      # 1. Build and push your Docker image
      - name: Build and Push Docker Image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: your-registry/your-app:${{ github.sha }}
          # Remember to update your deployment.yaml to use this new image tag!

      # 2. Use the TraceAssist Action to instrument your manifest
      - name: Instrument Kubernetes Manifest
        id: traceassist
        uses: your-github-username/traceassist-action@v1
        with:
          traceassist_host: 'https://api.your-traceassist-service.com'
          manifest_path: './k8s/deployment.yaml'
          deployment_name: 'my-production-app'

      # 3. Deploy your now-instrumented application
      - name: Deploy to Kubernetes
        uses: ahmadnassri/action-kubectl@v1
        with:
          command: apply -f ./k8s/deployment.yaml
        env:
          # You must configure this secret in your repository settings
          KUBE_CONFIG_DATA: ${{ secrets.KUBE_CONFIG_DATA }}
Action Inputs and OutputsInputsNameDescriptionRequiredtraceassist_hostThe URL of the hosted TraceAssist backend service.truemanifest_pathThe path to the Kubernetes manifest file to instrument.truedeployment_nameA unique name for your application deployment.trueOutputsNameDescriptionchanges_madeReturns "true" if the manifest was modified, otherwise "false".Advanced Usage: Committing Manifest ChangesYou can use the changes_made output to optionally commit the instrumented manifest back to your repository. This is useful for keeping your Git repository as the source of truth for your deployments.      # ... previous steps ...

      - name: Instrument Kubernetes Manifest
        id: traceassist
        uses: your-github-username/traceassist-action@v1
        with:
          traceassist_host: 'https://api.your-traceassist-service.com'
          manifest_path: './k8s/deployment.yaml'
          deployment_name: 'my-production-app'

      # New step to commit changes if the manifest was modified
      - name: Commit and Push Instrumented Manifest
        if: steps.traceassist.outputs.changes_made == 'true'
        run: |
          git config --global user.name 'github-actions[bot]'
          git config --global user.email 'github-actions[bot]@users.noreply.github.com'
          git add ./k8s/deployment.yaml
          git commit -m "ci: Auto-instrument manifest via TraceAssist"
          git push

      # ... deployment step ...
📜 LicenseThis GitHub Action is licensed under the MIT License.