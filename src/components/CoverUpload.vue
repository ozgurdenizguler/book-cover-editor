<template>
  <div>
    <input type="file" accept="image/*" @change="handleFileUpload" style="margin-bottom: 10%" />
    <div class="row-container">
      <div class="text-fields" v-if="imageUrl">
        <v-row>
          <v-col cols="12">
            <v-text-field
              v-model.number="fontSize"
              label="Font Size"
              type="number"
              @input="updateTextProperties"
              :disabled="!isTextSelected"
            />
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12">
            <v-select
              v-model="fontColor"
              :items="colorOptions"
              label="Font Color"
              @change="updateTextProperties"
              :disabled="!isTextSelected"
            ></v-select>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12">
            <v-text-field
              v-model.number="charSpacing"
              label="Char Spacing"
              type="number"
              :min="0"
              :step="1"
              @input="updateTextProperties"
              :disabled="!isTextSelected"
            />
          </v-col>
        </v-row>
      </div>
      <div class="canvas-container">
        <div ref="canvasContainer" style="position: relative">
          <canvas
            ref="canvas"
            style="position: absolute; top: 0; left: 0"
          ></canvas>
        </div>
      </div>
    </div>
    <div class="button-container">
      <v-btn color="primary" @click="goBack">Go Back</v-btn>
      <v-btn color="secondary" @click="openModal" :disabled="!imageUrl"
        >Save</v-btn
      >
    </div>
    <!-- Modal -->
    <v-dialog v-model="modalOpen" max-width="800" persistent>
      <v-card>
        <v-card-title class="headline grey lighten-2">
          <span>Preview the Cover</span>
        </v-card-title>
        <v-card-text class="d-flex justify-center">
          <img
            :src="editedImageUrl"
            alt="Edited Image"
            style="max-width: calc(100% - 40px); max-height: calc(100% - 64px)"
          />
        </v-card-text>
        <v-card-actions>
          <div>
            <v-btn color="primary" @click="modalOpen = false"> Back</v-btn>
          </div>
          <div class="flex-grow-1 text-right">
            <v-btn color="secondary" @click="downloadEditedImage"
              >Download</v-btn
            >
          </div>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import { fabric } from "fabric";

export default {
  name: "CoverUpload",
  props: {
    selectedBook: {
      type: Object,
      default: () => ({ title: "", author: "" }),
    },
  },
  data() {
    return {
      canvas: null,
      canvasWidth: 0,
      canvasHeight: 0,
      imageUrl: null,
      editedImageUrl: null,
      textObjects: [],
      fontSize: 15,
      fontColor: "red",
      charSpacing: 0,
      selectedTextObject: null,
      modalOpen: false,
      isTextSelected: false,
      colorOptions: ["red", "blue", "green", "yellow", "black", "orange"],
    };
  },
  mounted() {
    this.canvas = new fabric.Canvas(this.$refs.canvas);
  },
  watch: {
    fontColor: function () {
      if (this.selectedTextObject) {
        this.updateTextProperties();
      }
    },
  },
  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => {
          this.imageUrl = reader.result;
          this.updateCanvasSize();
          this.canvas.clear();
          this.addImageToCanvas();
        };
      }
    },
    updateCanvasSize() {
      const container = this.$refs.canvasContainer;
      const img = new Image();
      img.src = this.imageUrl;
      img.onload = () => {
        const aspectRatio = img.width / img.height;
        const containerWidth = container.clientWidth;
        const containerHeight = containerWidth / aspectRatio;
        this.canvasWidth = containerWidth;
        this.canvasHeight = containerHeight;
        this.canvas.setDimensions({
          width: this.canvasWidth,
          height: this.canvasHeight,
        });
      };
    },
    addImageToCanvas() {
      fabric.Image.fromURL(this.imageUrl, (img) => {
        img.scaleToWidth(this.canvasWidth);
        img.scaleToHeight(this.canvasHeight);
        img.set("selectable", false);
        this.canvas.add(img);
        this.addTextToCanvas();
      });
    },
    addTextToCanvas() {
      const titleText = new fabric.Text(this.selectedBook.title, {
        left: 10,
        top: 50,
        fontSize: this.fontSize,
        fill: this.fontColor,
        selectable: true,
        charSpacing: this.charSpacing,
      });
      const authorText = new fabric.Text(this.selectedBook.author, {
        left: 50,
        top: 100,
        fontSize: this.fontSize,
        fill: this.fontColor,
        selectable: true,
        charSpacing: this.charSpacing,
      });

      const enableTextFields = () => {
        this.isTextSelected = true;
      };

      const disableTextFields = () => {
        this.isTextSelected = false;
      };

      titleText.on("selected", () => {
        this.selectedTextObject = titleText;
        this.fontSize = titleText.fontSize;
        this.fontColor = titleText.fill;
        this.charSpacing = titleText.charSpacing;
        enableTextFields();
      });

      authorText.on("selected", () => {
        this.selectedTextObject = authorText;
        this.fontSize = authorText.fontSize;
        this.fontColor = authorText.fill;
        this.charSpacing = authorText.charSpacing;
        enableTextFields();
      });

      this.canvas.on("selection:cleared", () => {
        this.selectedTextObject = null;
        disableTextFields();
      });

      this.canvas.add(titleText);
      this.canvas.add(authorText);
      this.textObjects.push(titleText, authorText);

      this.canvas.on("object:moving", (e) => {
        const obj = e.target;
        if (obj.type === "text") {
          if (obj.left < 0) obj.left = 0;
          if (obj.top < 0) obj.top = 0;
          if (obj.left + obj.width * obj.scaleX > this.canvasWidth) {
            obj.left = this.canvasWidth - obj.width * obj.scaleX;
          }
          if (obj.top + obj.height * obj.scaleY > this.canvasHeight) {
            obj.top = this.canvasHeight - obj.height * obj.scaleY;
          }
          obj.setCoords();
          this.canvas.renderAll();
        }
      });
    },
    updateTextProperties() {
      if (this.selectedTextObject) {
        let newCharSpacing = this.charSpacing;
        if (newCharSpacing < 0) {
          newCharSpacing = 0;
        } else if (newCharSpacing > 5) {
          newCharSpacing = 5;
        }
        this.charSpacing = newCharSpacing;
        this.selectedTextObject.set({
          fontSize: this.fontSize,
          fill: this.fontColor,
          charSpacing: this.charSpacing * 100,
        });
        this.canvas.renderAll();
      }
    },
    openModal() {
      this.modalOpen = true;
      this.editedImageUrl = this.canvas.toDataURL({
        format: "jpeg",
        quality: 1,
      });
    },
    downloadEditedImage() {
      const link = document.createElement("a");
      link.href = this.editedImageUrl;
      link.download = "edited_image.jpeg";
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      this.modalOpen = false;
    },
    goBack() {
      this.$emit("backClicked");
    },
  },
};
</script>

<style scoped>
.button-container {
  display: flex;
  margin-top: 5%;
  flex-direction: row;
  justify-content: space-around;
  align-items: center;
}

.row-container {
  display: flex;
  justify-content: space-between;
}

.text-fields {
  width: 40%;
}

.canvas-container {
  width: 55%;
}

.v-dialog--active .v-dialog__content {
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
