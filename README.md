import { Component, ChangeDetectorRef } from '@angular/core';
import { bootstrapApplication } from '@angular/platform-browser';
import { provideZonelessChangeDetection } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="app">
      <header class="topbar">
        <div class="brand"><span class="logo">W</span> WallStudio</div>
        <nav>
          <button (click)="page='Home'">Home</button>
          <button (click)="page='Upload'">Visualizer</button>
          <button (click)="page='My Designs'">My Designs</button>
          <button (click)="page='About'">About</button>
        </nav>
        <button class="secondary" (click)="page='Login'">Sign In</button>
      </header>

      <main>
        <!-- HOME -->
        <section class="hero" *ngIf="page==='Home'">
          <div>
            <span class="eyebrow">YOUR HOME, YOUR COLOURS</span>
            <h1>See your walls in a <span>whole new colour.</span></h1>
            <p>Upload your room photo, select your walls and explore beautiful paint colours.</p>
            <button class="primary" (click)="page='Upload'">
              Start Visualizing →
            </button>
          </div>
          <div class="hero-room">
            <div class="fake-window"></div>
            <div class="fake-sofa"></div>
            <span>🌿</span>
          </div>
        </section>

        <section class="content" *ngIf="page==='Home'">
          <span class="eyebrow">GET INSPIRED</span>
          <h2>Find your perfect shade</h2>
          <div class="colours">
            <div *ngFor="let c of colors" class="colour-card">
              <div class="swatch" [style.background]="c.hex"></div>
              <strong>{{c.name}}</strong>
              <small>{{c.hex}}</small>
            </div>
          </div>
        </section>

        <!-- UPLOAD -->
        <section class="page" *ngIf="page==='Upload'">
          <span class="eyebrow">STEP 01</span>
          <h1>Upload your room photo</h1>
          <p>Choose a JPG or PNG image, maximum 10 MB.</p>

          <div class="upload-box">
            <input type="file"
              accept="image/jpeg,image/png"
              (change)="onFileSelected($event)">

            <div *ngIf="previewUrl">
              <h3>Your room photograph</h3>
              <img class="preview" [src]="previewUrl" alt="Room preview">
              <p>{{selectedFileName}}</p>
              <button class="secondary" (click)="removeImage()">Remove image</button>
              <button class="primary" (click)="openSelection()">
                Select Wall →
              </button>
            </div>
          </div>
          <p class="error" *ngIf="uploadError">{{uploadError}}</p>
        </section>

        <!-- WALL SELECTION -->
        <section class="page" *ngIf="page==='Selection'">
          <span class="eyebrow">STEP 02</span>
          <h1>Select the wall area</h1>
          <p>Click three or more points around the wall boundary.</p>

          <div class="editor" *ngIf="previewUrl && imageWidth">
            <svg class="wall-editor"
              [attr.viewBox]="'0 0 '+imageWidth+' '+imageHeight"
              (click)="addPoint($event)">

              <image [attr.href]="previewUrl"
                x="0" y="0"
                [attr.width]="imageWidth"
                [attr.height]="imageHeight"
                preserveAspectRatio="none">
              </image>

              <polygon *ngIf="selectionPoints.length>=3"
                [attr.points]="polygonPoints"
                fill="rgba(50,180,100,0.35)"
                stroke="#00a65a" stroke-width="5">
              </polygon>

              <polyline *ngIf="selectionPoints.length>0 && !selectionClosed"
                [attr.points]="polygonPoints"
                fill="none" stroke="#ffcc00" stroke-width="5">
              </polyline>

              <circle *ngFor="let point of selectionPoints"
                [attr.cx]="point.x" [attr.cy]="point.y"
                r="8" fill="#ffcc00" stroke="white" stroke-width="3">
              </circle>
            </svg>
          </div>

          <p class="instruction">Points selected: {{selectionPoints.length}}</p>

          <div class="toolbar">
            <button class="secondary" (click)="undoPoint()"
              [disabled]="selectionPoints.length===0 || selectionClosed">
              ↶ Undo
            </button>
            <button class="secondary" (click)="clearSelection()">Clear</button>
            <button class="primary" (click)="finishSelection()"
              [disabled]="selectionPoints.length<3 || selectionClosed">
              Finish Selection
            </button>
          </div>

          <div class="success-message" *ngIf="selectionClosed">
            ✓ Wall selected successfully!
            <br>
            <button class="primary" (click)="page='Paint'">
              Continue to Paint Colours →
            </button>
          </div>

          <button class="secondary" (click)="page='Upload'">← Back to Upload</button>
        </section>

        <!-- PAINT EDITOR -->
        <section class="page" *ngIf="page==='Paint'">
          <span class="eyebrow">STEP 03 — PAINT YOUR WALL</span>
          <h1>Choose your perfect colour</h1>
          <p>Select a shade below. It will be applied inside your selected wall area.</p>

          <div class="paint-editor" *ngIf="previewUrl && selectionClosed">
            <div class="paint-image">
              <svg
                [attr.viewBox]="'0 0 '+imageWidth+' '+imageHeight"
                class="paint-svg">

                <defs>
                  <clipPath id="wallMask">
                    <polygon [attr.points]="polygonPoints"></polygon>
                  </clipPath>
                </defs>

                <image
                  [attr.href]="previewUrl"
                  x="0" y="0"
                  [attr.width]="imageWidth"
                  [attr.height]="imageHeight"
                  preserveAspectRatio="none">
                </image>

                <rect
                  x="0" y="0"
                  [attr.width]="imageWidth"
                  [attr.height]="imageHeight"
                  [attr.fill]="selectedColor"
                  [attr.opacity]="opacity / 100"
                  clip-path="url(#wallMask)">
                </rect>

                <polygon
                  [attr.points]="polygonPoints"
                  fill="none"
                  stroke="#ffffff"
                  stroke-width="3"
                  stroke-dasharray="10 6">
                </polygon>
              </svg>
            </div>

            <div class="paint-controls">
              <h2>Paint colour palette</h2>

              <div class="paint-palette">
                <button
                  *ngFor="let c of colors"
                  class="paint-swatch"
                  [class.selected]="selectedColor===c.hex"
                  [style.background]="c.hex"
                  [title]="c.name"
                  (click)="selectColor(c.hex,c.name)">
                </button>
              </div>

              <div class="selected-info">
                <div class="selected-preview"
                  [style.background]="selectedColor">
                </div>
                <div>
                  <strong>{{selectedColorName}}</strong>
                  <small>{{selectedColor}}</small>
                </div>
              </div>

              <label class="opacity-label">
                Paint opacity: {{opacity}}%
              </label>

              <input type="range"
                min="10" max="100" step="5"
                [value]="opacity"
                (input)="changeOpacity($event)"
                class="opacity-slider">

              <div class="range-labels">
                <span>Transparent</span>
                <span>Solid</span>
              </div>

              <button class="secondary" (click)="resetPaint()">
                Reset Paint
              </button>

              <button class="primary" (click)="page='Selection'">
                Edit Wall Selection
              </button>

              <button class="primary download-button" (click)="downloadImage()">
                ↓ Download Painted Room PNG
              </button>
            </div>
          </div>

          <!-- BEFORE AND AFTER -->
          <div class="comparison-section"
               *ngIf="previewUrl && selectionClosed">

            <h2>Before & After Comparison</h2>
            <p>Drag the slider to compare your original and painted wall.</p>

            <div class="comparison-stage"
                 [style.aspect-ratio]="imageWidth + ' / ' + imageHeight">

              <svg class="comparison-svg"
                [attr.viewBox]="'0 0 '+imageWidth+' '+imageHeight">

                <defs>
                  <clipPath id="comparisonWallMask">
                    <polygon [attr.points]="polygonPoints"></polygon>
                  </clipPath>
                </defs>

                <image [attr.href]="previewUrl"
                  x="0" y="0"
                  [attr.width]="imageWidth"
                  [attr.height]="imageHeight">
                </image>

                <rect x="0" y="0"
                  [attr.width]="imageWidth"
                  [attr.height]="imageHeight"
                  [attr.fill]="selectedColor"
                  [attr.opacity]="opacity / 100"
                  clip-path="url(#comparisonWallMask)">
                </rect>
              </svg>

              <img class="comparison-before"
                [src]="previewUrl"
                [style.clip-path]="'inset(0 '+(100-comparisonPosition)+'% 0 0)'">

              <span class="comparison-label before-label">BEFORE</span>
              <span class="comparison-label after-label">AFTER</span>

              <div class="comparison-divider"
                [style.left.%]="comparisonPosition">
                <div class="comparison-handle">↔</div>
              </div>

              <input type="range"
                min="0" max="100"
                [value]="comparisonPosition"
                (input)="changeComparison($event)"
                class="comparison-range">
            </div>

            <p class="comparison-hint">
              ← Drag left or right to compare your design →
            </p>
          </div>

          <div class="notice">
            Note: The preview is an approximation. Actual paint appearance
            may vary with room lighting, wall texture and screen calibration.
          </div>
        </section>

        <!-- OTHER PAGES -->
        <section class="page" *ngIf="page==='My Designs'">
          <h1>My Designs</h1>
          <p>Saved designs will be available after database integration.</p>
        </section>

        <section class="page" *ngIf="page==='About'">
          <h1>About WallStudio</h1>
          <p>Preview wall paint colours on your own room photographs.</p>
        </section>

        <section class="page" *ngIf="page==='Login'">
          <h1>Sign In</h1>
          <p>Secure login will be developed in a later step.</p>
        </section>
      </main>

      <footer>
        <strong>WallStudio</strong>
        <span>Imagine it. Colour it. Love it.</span>
      </footer>
    </div>
  `,

  styles: [`
    * { box-sizing:border-box; }

    .app {
      min-height:100vh;
      background:#faf9f6;
      color:#282a25;
      font-family:Arial,sans-serif;
    }

    .topbar {
      display:flex;
      justify-content:space-between;
      align-items:center;
      flex-wrap:wrap;
      gap:15px;
      padding:18px 7%;
      background:white;
      border-bottom:1px solid #eee;
    }

    .brand { font-size:21px;font-weight:bold; }

    .logo {
      background:#536c50;color:white
