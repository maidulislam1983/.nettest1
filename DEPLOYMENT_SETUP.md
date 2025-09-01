# Azure DevOps Pipeline Setup for Automated Deployment

This repository contains two Azure Pipeline configurations for automating the deployment of your ASP.NET Core application using your existing publish profiles.

## Pipeline Options

### 1. Full CI/CD Pipeline (`azure-pipelines.yml`)
- **Features**: Multi-stage pipeline with separate Build and Deploy stages
- **Benefits**: Better for production environments with approval gates and environments
- **Includes**: Build artifacts, testing, environment protection

### 2. Simple Pipeline (`azure-pipelines-simple.yml`)
- **Features**: Single-stage pipeline that directly uses publish profiles
- **Benefits**: Simpler setup, faster execution
- **Best for**: Development/staging environments or simple deployments

## Setup Instructions

### Step 1: Configure Pipeline Variables

In your Azure DevOps project, add these secure variables:

#### For Web Deploy (MSDeploy):
- `DeployUsername`: `saidul1983-001`
- `DeployPassword`: Your hosting provider password (mark as secret)

#### For FTP Deploy (if using FTP option):
- `FtpUsername`: `saidul1983-001`
- `FtpPassword`: Your FTP password (mark as secret)

### Step 2: Create Environment (for full pipeline only)

1. Go to **Pipelines** → **Environments**
2. Create new environment named `Production`
3. Add approval gates if needed
4. Configure any additional security settings

### Step 3: Choose Your Pipeline

#### Option A: Use the Full Pipeline
1. Rename `azure-pipelines.yml` to keep it as your main pipeline
2. The pipeline will automatically trigger on commits to `main` branch
3. Build stage runs first, then deployment stage (with approvals if configured)

#### Option B: Use the Simple Pipeline
1. Rename `azure-pipelines-simple.yml` to `azure-pipelines.yml`
2. Delete the complex pipeline file
3. This will deploy directly after build/test

### Step 4: Verify Publish Profile Settings

Your publish profiles are located in:
- `WebApplication1/Properties/PublishProfiles/`

Current profiles:
- **Web Deploy**: `saidul1983-001-site1 - Web Deploy.pubxml`
- **FTP**: `saidul1983-001-site1 - FTP.pubxml`

Target URL: `http://saidul1983-001-site1.mtempurl.com/`

## Pipeline Features

### Build Stage
- ✅ .NET 8 SDK installation
- ✅ Package restoration
- ✅ Solution build
- ✅ Unit test execution (if tests exist)
- ✅ Application publishing
- ✅ Artifact creation

### Deploy Stage
- ✅ Artifact download
- ✅ Web Deploy (MSDeploy) deployment
- ✅ Alternative FTP deployment option
- ✅ App offline during deployment
- ✅ Remove additional files option

## Troubleshooting

### Common Issues

1. **Authentication Failures**
   - Verify username/password variables are set correctly
   - Check that passwords are marked as "secret"
   - Ensure hosting provider credentials are valid

2. **Web Deploy Issues**
   - If Web Deploy fails, uncomment the FTP deployment section
   - Check firewall settings on hosting provider
   - Verify MSDeploy service is available

3. **Build Failures**
   - Ensure all NuGet packages are available
   - Check .NET version compatibility
   - Verify project references are correct

### Alternative Deployment Methods

If the automated deployment fails, you can also:

1. **Manual Web Deploy**: Use the publish profile directly from Visual Studio
2. **FTP Upload**: Switch to FTP deployment in the pipeline
3. **Package Download**: Download build artifacts and deploy manually

## Security Best Practices

1. **Never commit passwords** to source control
2. **Use Azure Key Vault** for sensitive data in production
3. **Enable approval gates** for production deployments
4. **Limit pipeline permissions** to necessary resources only
5. **Monitor deployment logs** for security issues

## Customization Options

### Adding More Environments
```yaml
- stage: DeployStaging
  displayName: 'Deploy to Staging'
  dependsOn: Build
  condition: eq(variables['Build.SourceBranch'], 'refs/heads/develop')
```

### Adding Database Migrations
```yaml
- task: DotNetCoreCLI@2
  displayName: 'Run database migrations'
  inputs:
    command: 'custom'
    custom: 'ef'
    arguments: 'database update --project $(projectPath)'
```

### Adding Health Checks
```yaml
- task: PowerShell@2
  displayName: 'Verify deployment'
  inputs:
    targetType: 'inline'
    script: |
      $response = Invoke-WebRequest -Uri "http://saidul1983-001-site1.mtempurl.com/health" -UseBasicParsing
      if ($response.StatusCode -ne 200) { exit 1 }
```

## Support

If you encounter issues:
1. Check the pipeline logs in Azure DevOps
2. Verify hosting provider status
3. Test publish profiles manually from Visual Studio
4. Contact your hosting provider for deployment issues
